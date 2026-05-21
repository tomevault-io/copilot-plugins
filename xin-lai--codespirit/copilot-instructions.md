## performance

> CodeSpirit 性能优化规范 - 异步编程、缓存策略、查询优化、分布式场景


# 性能优化规范

## 异步编程
- **所有 I/O 操作必须使用异步方法**（`async/await`）
- **避免阻塞调用**：禁止使用 `Task.Result` 和 `Task.Wait()`
- **高频调用优化**：使用 `ValueTask<T>` 减少堆分配
- **避免异步循环**：使用批量处理代替循环中的异步操作

### 正确示例
```csharp
// ✅ 正确：使用异步方法
public async Task<List<QuestionDto>> GetQuestionsAsync(QuestionQueryDto query)
{
    var entities = await _repository.GetListAsync(query);
    return _mapper.Map<List<QuestionDto>>(entities);
}

// ✅ 正确：批量处理
public async Task<List<QuestionDto>> GetQuestionsByIdsAsync(List<long> ids)
{
    // 一次查询获取所有数据
    var entities = await _dbContext.Questions
        .Where(q => ids.Contains(q.Id))
        .ToListAsync();
    return _mapper.Map<List<QuestionDto>>(entities);
}
```

### 错误示例
```csharp
// ❌ 错误：阻塞调用
public List<QuestionDto> GetQuestions(QuestionQueryDto query)
{
    var entities = _repository.GetListAsync(query).Result;  // 阻塞！
    return _mapper.Map<List<QuestionDto>>(entities);
}

// ❌ 错误：异步循环
public async Task<List<QuestionDto>> GetQuestionsByIdsAsync(List<long> ids)
{
    var results = new List<QuestionDto>();
    foreach (var id in ids)
    {
        var entity = await _repository.GetByIdAsync(id);  // N 次查询！
        results.Add(_mapper.Map<QuestionDto>(entity));
    }
    return results;
}
```

## 缓存策略

### 缓存键命名
```csharp
// 格式：{service}:{entity}:{identifier}
"exam:question:123"
"exam:questions:list:categoryId_5"
"exam:user:profile:456"

// 租户缓存：{tenantId}:{service}:{entity}:{identifier}
"tenant_1:exam:question:123"
```

### 缓存级别
- **L1 (内存缓存)**：频繁访问的小数据（配置、枚举等）
- **L2 (Redis)**：需要跨实例共享的数据
- **L1AndL2**：热点数据（如用户信息、权限数据）

### 使用示例
```csharp
public class QuestionService
{
    private readonly ICacheService _cacheService;
    
    public async Task<QuestionDto> GetByIdAsync(long id)
    {
        return await _cacheService.GetOrSetAsync(
            $"exam:question:{id}",
            async () => await _repository.GetByIdAsync(id),
            new CacheOptions 
            { 
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(30),
                Level = CacheLevel.L1AndL2
            });
    }
    
    public async Task UpdateAsync(long id, UpdateQuestionDto dto)
    {
        await _repository.UpdateAsync(id, dto);
        
        // 更新后清除缓存
        await _cacheService.RemoveAsync($"exam:question:{id}");
    }
}
```

### 过期策略
- **静态数据**：绝对过期时间（1小时以上）
- **动态数据**：滑动过期时间（5-30分钟）
- **实时数据**：不缓存或短期缓存（1-5分钟）

## EF Core 查询优化

### 1. AsNoTracking 只读查询
```csharp
// ✅ 只读查询使用 AsNoTracking
public async Task<List<QuestionDto>> GetListAsync(QuestionQueryDto query)
{
    var entities = await _dbContext.Questions
        .AsNoTracking()  // 不跟踪实体变化，提升性能
        .Where(q => q.IsDeleted == false)
        .ToListAsync();
    return _mapper.Map<List<QuestionDto>>(entities);
}
```

### 2. Include 避免 N+1 查询
```csharp
// ✅ 正确：一次查询加载关联数据
public async Task<List<QuestionDto>> GetListWithCategoryAsync()
{
    var entities = await _dbContext.Questions
        .Include(q => q.Category)  // 预加载关联数据
        .AsNoTracking()
        .ToListAsync();
    return _mapper.Map<List<QuestionDto>>(entities);
}

// ❌ 错误：N+1 查询
public async Task<List<QuestionDto>> GetListWithCategoryAsync()
{
    var entities = await _dbContext.Questions.ToListAsync();
    foreach (var entity in entities)
    {
        entity.Category = await _dbContext.Categories.FindAsync(entity.CategoryId);  // N次查询！
    }
    return _mapper.Map<List<QuestionDto>>(entities);
}
```

### 3. AsSplitQuery 处理笛卡尔积
```csharp
// ✅ 多对多关联使用 AsSplitQuery
public async Task<ExamDto> GetExamWithQuestionsAsync(long examId)
{
    var exam = await _dbContext.Exams
        .Include(e => e.Questions)
        .ThenInclude(q => q.Options)
        .AsSplitQuery()  // 拆分为多个查询，避免笛卡尔积
        .FirstOrDefaultAsync(e => e.Id == examId);
    return _mapper.Map<ExamDto>(exam);
}
```

### 4. 批量操作
```csharp
// ✅ 批量更新（EF Core 7+）
public async Task UpdateScoresAsync(Dictionary<long, decimal> scores)
{
    await _dbContext.Questions
        .Where(q => scores.Keys.Contains(q.Id))
        .ExecuteUpdateAsync(setters => setters
            .SetProperty(q => q.Score, q => scores[q.Id]));
}

// ✅ 批量删除（EF Core 7+）
public async Task DeleteByCategoryAsync(long categoryId)
{
    await _dbContext.Questions
        .Where(q => q.CategoryId == categoryId)
        .ExecuteDeleteAsync();
}
```

### 5. 投影查询
```csharp
// ✅ 只查询需要的字段
public async Task<List<QuestionListItemDto>> GetListAsync()
{
    return await _dbContext.Questions
        .Select(q => new QuestionListItemDto
        {
            Id = q.Id,
            Content = q.Content,
            CategoryName = q.Category.Name
        })
        .ToListAsync();
}
```

## 分布式场景优化

### 1. 分布式锁
```csharp
public async Task<bool> TryStartExamAsync(long examId, long userId)
{
    var lockKey = $"exam:start:{examId}:{userId}";
    
    await using var lockHandle = await _distributedLock.TryAcquireAsync(
        lockKey, 
        TimeSpan.FromSeconds(10));
    
    if (lockHandle == null)
    {
        throw new BusinessException("Errors.ExamAlreadyStarted");
    }
    
    // 执行业务逻辑
    await _examService.StartExamAsync(examId, userId);
    return true;
}
```

### 2. 分布式缓存
```csharp
public async Task<UserDto> GetUserAsync(long userId)
{
    return await _cacheService.GetOrSetAsync(
        $"user:{userId}",
        async () => await _userRepository.GetByIdAsync(userId),
        new CacheOptions 
        { 
            Level = CacheLevel.L2  // Redis 分布式缓存
        });
}
```

### 3. 事件驱动解耦
```csharp
// 发布事件
public async Task CreateOrderAsync(CreateOrderDto dto)
{
    var order = await _orderRepository.CreateAsync(dto);
    
    // 发布事件，异步处理后续逻辑
    await _eventBus.PublishAsync(new OrderCreatedEvent
    {
        OrderId = order.Id,
        UserId = order.UserId
    });
}

// 订阅事件
public class OrderCreatedEventHandler : IEventHandler<OrderCreatedEvent>
{
    public async Task HandleAsync(OrderCreatedEvent @event)
    {
        // 异步处理：发送通知、更新库存等
        await _notificationService.NotifyOrderCreatedAsync(@event.OrderId);
    }
}
```

### 4. 最终一致性
```csharp
// 避免分布式事务，采用最终一致性
public async Task CreateOrderAsync(CreateOrderDto dto)
{
    // 1. 先创建订单（主业务）
    var order = await _orderRepository.CreateAsync(dto);
    
    // 2. 发布事件，异步处理库存扣减
    await _eventBus.PublishAsync(new OrderCreatedEvent
    {
        OrderId = order.Id,
        Items = dto.Items
    });
    
    // 库存扣减失败时，通过补偿机制处理
}
```

## 数据库连接池
- 使用连接池管理数据库连接
- 避免长时间占用连接
- 及时释放连接（使用 `using` 或 DI 容器管理）

## 注意事项
- 定期监控性能指标（响应时间、吞吐量、资源使用）
- 使用 Application Insights 或类似工具进行性能分析
- 避免过早优化，先测量后优化
- 考虑分布式场景下的数据一致性
- 缓存要考虑缓存穿透、雪崩、击穿等问题

---
> Source: [xin-lai/CodeSpirit](https://github.com/xin-lai/CodeSpirit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
