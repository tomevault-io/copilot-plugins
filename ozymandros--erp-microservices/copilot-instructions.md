## erp-microservices

> DAPR service invocation and pub/sub communication rules


# Service Communication Rules

## DAPR HTTP Service Invocation

**ALWAYS use DAPR for service-to-service calls:**

```csharp
public class OrderService : IOrderService
{
    private readonly DaprClient _daprClient;

    public async Task<ProductDto?> GetProductFromInventory(int productId)
    {
        var httpClient = _daprClient.CreateInvokeHttpClient("inventory-service");
        var response = await httpClient.GetAsync($"/api/inventory/products/{productId}");
        
        if (!response.IsSuccessStatusCode) return null;
        
        return await response.Content.ReadFromJsonAsync<ProductDto>();
    }
}
```

**Service App-IDs:**

- `auth-service`
- `billing-service`
- `inventory-service`
- `orders-service`
- `purchasing-service`
- `sales-service`

## DAPR Pub/Sub Events

**Publishing events:**

```csharp
await _daprClient.PublishEventAsync(
    "pubsub",
    "order-created",
    new OrderCreatedEvent 
    { 
        OrderId = order.Id,
        CustomerId = order.CustomerId
    }
);
```

**Subscribing to events (in Program.cs):**

```csharp
app.MapPost("/order-created", async (
    OrderCreatedEvent @event,
    ILogger<Program> logger,
    INotificationService notificationService) =>
{
    logger.LogInformation("Received order-created event: {OrderId}", @event.OrderId);
    await notificationService.SendOrderConfirmationAsync(@event.OrderId);
    return Results.Ok();
});
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ozymandros)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Ozymandros)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
