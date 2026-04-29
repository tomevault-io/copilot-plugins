## harsh-hackathon-backend

> This guide details the implementation patterns for data transformation between different layers of the system.

# Data Transformation Guide

This guide details the implementation patterns for data transformation between different layers of the system.

## Core Concepts

Mappers handle data transformation between layers:

1. External data to domain entities
2. Domain entities to DTOs
3. Domain entities to infrastructure format
4. Isolate transformation logic
5. Keep domain entities clean
6. Error handling and validation
7. Type safety and consistency
8. Performance optimization
9. Caching support
10. Logging and monitoring

## Directory Structure

```
mappers/
├── orderMapper.ts
├── orderMapper.test.ts
├── paymentMapper.ts
├── paymentMapper.test.ts
├── customerMapper.ts
├── customerMapper.test.ts
├── cache/
│   ├── orderCache.ts
│   └── paymentCache.ts
└── utils/
    ├── validation.ts
    └── formatting.ts
```

## Mapper Best Practices

### Interface Guidelines

1. **Clear Contracts**: Define explicit input/output types
2. **Single Responsibility**: Each mapper handles one entity type
3. **Error Handling**: Define error types and handling strategies
4. **Type Safety**: Use proper types and interfaces
5. **Documentation**: Document all methods and parameters
6. **Testing**: Include test interfaces
7. **Validation**: Validate input data
8. **Caching**: Support caching strategies

### Implementation Guidelines

1. **Error Handling**: Implement proper error handling
2. **Logging**: Log important operations and errors
3. **Monitoring**: Monitor mapper performance
4. **Validation**: Validate all inputs
5. **Caching**: Implement caching strategies
6. **Performance**: Optimize transformation operations
7. **Testing**: Write comprehensive tests
8. **Documentation**: Document implementation details

### Performance Guidelines

1. **Caching**: Use caching for expensive transformations
2. **Batching**: Support batch transformations
3. **Lazy Loading**: Implement lazy loading where appropriate
4. **Memory Management**: Optimize memory usage
5. **Async Operations**: Support async transformations
6. **Streaming**: Support streaming for large datasets
7. **Validation**: Optimize validation logic
8. **Monitoring**: Monitor performance metrics

## Implementation Examples

### Basic Mapper

```typescript
// mappers/orderMapper.ts

import { Order } from '../domain/entities/order';
import { OrderDto } from '../dtos/orderDto';
import { Logger } from '../utils/logger';
import { CacheService } from '../services/cacheService';
import { MonitoringService } from '../services/monitoringService';
import { ValidationError } from '../utils/errors';
import { z } from 'zod';

const orderSchema = z.object({
  id: z.string(),
  customerId: z.string(),
  items: z.array(z.object({
    productId: z.string(),
    price: z.number().positive(),
    quantity: z.number().int().positive()
  })),
  status: z.enum(['pending', 'processing', 'shipped', 'delivered', 'cancelled']),
  total: z.number().nonnegative(),
  createdAt: z.date()
});

export class OrderMapper {
  private static readonly logger = new Logger('OrderMapper');
  private static readonly cacheService: CacheService;
  private static readonly monitoringService: MonitoringService;

  static async toDomain(raw: any): Promise<Order> {
    try {
      this.logger.info('Mapping raw data to Order domain entity');
      this.monitoringService.incrementCounter('order_mapping_to_domain');

      const cachedOrder = await this.cacheService.get<Order>(`order:${raw.id}`);
      if (cachedOrder) {
        return cachedOrder;
      }

      const orderProps = {
        id: raw.id || raw._id,
        customerId: raw.customer_id || raw.customerId,
        items: raw.items.map((item: any) => ({
          productId: item.product_id || item.productId,
          price: item.price,
          quantity: item.quantity
        })),
        status: raw.status || 'pending',
        total: raw.total || 0,
        createdAt: raw.created_at ? new Date(raw.created_at) : new Date()
      };

      orderSchema.parse(orderProps);
      
      const order = Order.create(orderProps);
      await this.cacheService.set(`order:${order.id}`, order, 3600); // Cache for 1 hour
      
      return order;
    } catch (error) {
      this.logger.error('Failed to map raw data to Order domain entity', { error });
      this.monitoringService.incrementCounter('order_mapping_to_domain_error');
      throw new ValidationError('Invalid order data', error);
    }
  }
  
  static async toDto(order: Order): Promise<OrderDto> {
    try {
      this.logger.info('Mapping Order domain entity to DTO');
      this.monitoringService.incrementCounter('order_mapping_to_dto');

      const cachedDto = await this.cacheService.get<OrderDto>(`order_dto:${order.id}`);
      if (cachedDto) {
        return cachedDto;
      }

      const orderDetails = order.toObject();
      
      const dto = {
        id: orderDetails.id,
        customerId: orderDetails.customerId,
        items: orderDetails.items.map(item => ({
          productId: item.productId,
          price: this.formatPrice(item.price),
          quantity: item.quantity,
          subtotal: this.formatPrice(item.price * item.quantity)
        })),
        status: this.mapStatus(orderDetails.status),
        total: this.formatPrice(orderDetails.total),
        createdAt: orderDetails.createdAt.toISOString()
      };

      await this.cacheService.set(`order_dto:${order.id}`, dto, 3600); // Cache for 1 hour
      
      return dto;
    } catch (error) {
      this.logger.error('Failed to map Order domain entity to DTO', { error });
      this.monitoringService.incrementCounter('order_mapping_to_dto_error');
      throw new ValidationError('Failed to map order to DTO', error);
    }
  }
  
  static async toInfrastructure(order: Order): Promise<any> {
    try {
      this.logger.info('Mapping Order domain entity to infrastructure format');
      this.monitoringService.incrementCounter('order_mapping_to_infrastructure');

      const orderDetails = order.toObject();
      
      return {
        _id: orderDetails.id,
        customer_id: orderDetails.customerId,
        items: orderDetails.items.map(item => ({
          product_id: item.productId,
          price: item.price,
          quantity: item.quantity
        })),
        status: orderDetails.status,
        total: orderDetails.total,
        created_at: orderDetails.createdAt
      };
    } catch (error) {
      this.logger.error('Failed to map Order domain entity to infrastructure format', { error });
      this.monitoringService.incrementCounter('order_mapping_to_infrastructure_error');
      throw new ValidationError('Failed to map order to infrastructure format', error);
    }
  }
  
  private static mapStatus(status: string): string {
    const statusMap: Record<string, string> = {
      'pending': 'Pending',
      'processing': 'Processing',
      'shipped': 'Shipped',
      'delivered': 'Delivered',
      'cancelled': 'Cancelled'
    };
    
    return statusMap[status] || 'Unknown';
  }
  
  private static formatPrice(price: number): string {
    return `$${price.toFixed(2)}`;
  }
}
```

### Advanced Mapper with Filtering and Caching

```typescript
// mappers/paymentMapper.ts

import { Payment } from '../domain/entities/payment';
import { PaymentDto } from '../dtos/paymentDto';
import { Logger } from '../utils/logger';
import { CacheService } from '../services/cacheService';
import { MonitoringService } from '../services/monitoringService';
import { ValidationError } from '../utils/errors';
import { z } from 'zod';

const paymentSchema = z.object({
  id: z.string(),
  orderId: z.string(),
  amount: z.number().positive(),
  currency: z.string().length(3),
  status: z.enum(['pending', 'completed', 'failed', 'refunded']),
  method: z.enum(['credit_card', 'bank_transfer', 'digital_wallet']),
  transactionId: z.string(),
  createdAt: z.date()
});

export interface PaymentFilterProps {
  minAmount?: number;
  maxAmount?: number;
  status?: string;
  dateRange?: {
    start: Date;
    end: Date;
  };
}

export class PaymentMapper {
  private static readonly logger = new Logger('PaymentMapper');
  private static readonly cacheService: CacheService;
  private static readonly monitoringService: MonitoringService;

  static async toDomain(raw: any): Promise<Payment> {
    try {
      this.logger.info('Mapping raw data to Payment domain entity');
      this.monitoringService.incrementCounter('payment_mapping_to_domain');

      const cachedPayment = await this.cacheService.get<Payment>(`payment:${raw.id}`);
      if (cachedPayment) {
        return cachedPayment;
      }

      const paymentProps = {
        id: raw.id,
        orderId: raw.order_id || raw.orderId,
        amount: raw.amount,
        currency: raw.currency || 'USD',
        status: raw.status || 'pending',
        method: raw.method || 'credit_card',
        transactionId: raw.transaction_id || raw.transactionId,
        createdAt: raw.created_at ? new Date(raw.created_at) : new Date()
      };

      paymentSchema.parse(paymentProps);
      
      const payment = Payment.create(paymentProps);
      await this.cacheService.set(`payment:${payment.id}`, payment, 3600); // Cache for 1 hour
      
      return payment;
    } catch (error) {
      this.logger.error('Failed to map raw data to Payment domain entity', { error });
      this.monitoringService.incrementCounter('payment_mapping_to_domain_error');
      throw new ValidationError('Invalid payment data', error);
    }
  }
  
  static async toDto(payment: Payment): Promise<PaymentDto> {
    try {
      this.logger.info('Mapping Payment domain entity to DTO');
      this.monitoringService.incrementCounter('payment_mapping_to_dto');

      const cachedDto = await this.cacheService.get<PaymentDto>(`payment_dto:${payment.id}`);
      if (cachedDto) {
        return cachedDto;
      }

      const paymentDetails = payment.toObject();
      
      const dto = {
        id: paymentDetails.id,
        orderId: paymentDetails.orderId,
        amount: this.formatAmount(paymentDetails.amount, paymentDetails.currency),
        status: this.mapStatus(paymentDetails.status),
        method: this.mapMethod(paymentDetails.method),
        transactionId: paymentDetails.transactionId,
        createdAt: paymentDetails.createdAt.toISOString()
      };

      await this.cacheService.set(`payment_dto:${payment.id}`, dto, 3600); // Cache for 1 hour
      
      return dto;
    } catch (error) {
      this.logger.error('Failed to map Payment domain entity to DTO', { error });
      this.monitoringService.incrementCounter('payment_mapping_to_dto_error');
      throw new ValidationError('Failed to map payment to DTO', error);
    }
  }
  
  static async filterPayments(payments: Payment[], filters: PaymentFilterProps): Promise<Payment[]> {
    try {
      this.logger.info('Filtering payments');
      this.monitoringService.incrementCounter('payment_filtering');

      const cacheKey = `payment_filter:${JSON.stringify(filters)}`;
      const cachedResult = await this.cacheService.get<Payment[]>(cacheKey);
      if (cachedResult) {
        return cachedResult;
      }

      const filteredPayments = payments.filter(payment => {
        const details = payment.toObject();
        
        if (filters.minAmount && details.amount < filters.minAmount) {
          return false;
        }
        
        if (filters.maxAmount && details.amount > filters.maxAmount) {
          return false;
        }
        
        if (filters.status && details.status !== filters.status) {
          return false;
        }
        
        if (filters.dateRange) {
          const paymentDate = details.createdAt;
          if (paymentDate < filters.dateRange.start || paymentDate > filters.dateRange.end) {
            return false;
          }
        }
        
        return true;
      });

      await this.cacheService.set(cacheKey, filteredPayments, 300); // Cache for 5 minutes
      
      return filteredPayments;
    } catch (error) {
      this.logger.error('Failed to filter payments', { error });
      this.monitoringService.incrementCounter('payment_filtering_error');
      throw new ValidationError('Failed to filter payments', error);
    }
  }
  
  private static mapStatus(status: string): string {
    const statusMap: Record<string, string> = {
      'pending': 'Pending',
      'completed': 'Completed',
      'failed': 'Failed',
      'refunded': 'Refunded'
    };
    
    return statusMap[status] || 'Unknown';
  }
  
  private static mapMethod(method: string): string {
    const methodMap: Record<string, string> = {
      'credit_card': 'Credit Card',
      'bank_transfer': 'Bank Transfer',
      'digital_wallet': 'Digital Wallet'
    };
    
    return methodMap[method] || 'Unknown';
  }
  
  private static formatAmount(amount: number, currency: string): string {
    return new Intl.NumberFormat('en-US', {
      style: 'currency',
      currency: currency
    }).format(amount);
  }
}
```

### Mapper Test

```typescript
// mappers/orderMapper.test.ts

import { OrderMapper } from './orderMapper';
import { Order } from '../domain/entities/order';
import { CacheService } from '../services/cacheService';
import { MonitoringService } from '../services/monitoringService';
import { ValidationError } from '../utils/errors';

jest.mock('../domain/entities/order');
jest.mock('../services/cacheService');
jest.mock('../services/monitoringService');

describe('OrderMapper', () => {
  let mockCache: jest.Mocked<CacheService>;
  let mockMonitoring: jest.Mocked<MonitoringService>;
  
  beforeEach(() => {
    mockCache = {
      get: jest.fn(),
      set: jest.fn(),
      del: jest.fn()
    } as unknown as jest.Mocked<CacheService>;
    
    mockMonitoring = {
      incrementCounter: jest.fn()
    } as unknown as jest.Mocked<MonitoringService>;
    
    (Order.create as jest.Mock).mockImplementation((props) => ({
      toObject: () => props,
      id: props.id
    }));
  });

  describe('toDomain', () => {
    it('should map raw data with standard fields', async () => {
      const rawData = {
        id: 'ORD-123',
        customerId: 'CUST-456',
        items: [
          { productId: 'PROD-1', price: 10, quantity: 2 },
          { productId: 'PROD-2', price: 20, quantity: 1 }
        ],
        status: 'pending',
        total: 40,
        createdAt: '2023-01-01T00:00:00.000Z'
      };

      mockCache.get.mockResolvedValueOnce(null);

      const result = await OrderMapper.toDomain(rawData);
      
      expect(Order.create).toHaveBeenCalledWith({
        id: 'ORD-123',
        customerId: 'CUST-456',
        items: [
          { productId: 'PROD-1', price: 10, quantity: 2 },
          { productId: 'PROD-2', price: 20, quantity: 1 }
        ],
        status: 'pending',
        total: 40,
        createdAt: expect.any(Date)
      });

      expect(mockCache.set).toHaveBeenCalled();
      expect(mockMonitoring.incrementCounter).toHaveBeenCalledWith('order_mapping_to_domain');
    });

    it('should return cached order if available', async () => {
      const rawData = { id: 'ORD-123' };
      const cachedOrder = { id: 'ORD-123', toObject: () => ({ id: 'ORD-123' }) };

      mockCache.get.mockResolvedValueOnce(cachedOrder);

      const result = await OrderMapper.toDomain(rawData);

      expect(result).toBe(cachedOrder);
      expect(Order.create).not.toHaveBeenCalled();
      expect(mockCache.set).not.toHaveBeenCalled();
    });

    it('should throw ValidationError for invalid data', async () => {
      const rawData = {
        id: 'ORD-123',
        items: [{ price: -10 }] // Invalid price
      };

      mockCache.get.mockResolvedValueOnce(null);

      await expect(OrderMapper.toDomain(rawData))
        .rejects
        .toThrow(ValidationError);

      expect(mockMonitoring.incrementCounter)
        .toHaveBeenCalledWith('order_mapping_to_domain_error');
    });
  });

  describe('toDto', () => {
    it('should map order to DTO', async () => {
      const order = {
        id: 'ORD-123',
        toObject: () => ({
          id: 'ORD-123',
          customerId: 'CUST-456',
          items: [
            { productId: 'PROD-1', price: 10, quantity: 2 }
          ],
          status: 'pending',
          total: 20,
          createdAt: new Date()
        })
      };

      mockCache.get.mockResolvedValueOnce(null);

      const result = await OrderMapper.toDto(order as Order);

      expect(result).toEqual({
        id: 'ORD-123',
        customerId: 'CUST-456',
        items: [
          {
            productId: 'PROD-1',
            price: '$10.00',
            quantity: 2,
            subtotal: '$20.00'
          }
        ],
        status: 'Pending',
        total: '$20.00',
        createdAt: expect.any(String)
      });

      expect(mockCache.set).toHaveBeenCalled();
      expect(mockMonitoring.incrementCounter).toHaveBeenCalledWith('order_mapping_to_dto');
    });
  });
});
```

## Testing Approach for Mappers

The mapper tests focus on verifying data transformation logic and error handling.

### Testing Guidelines

1. **Unit Tests**: Test mapper methods in isolation
2. **Integration Tests**: Test with actual data
3. **Error Handling**: Test error scenarios
4. **Validation**: Test input validation
5. **Caching**: Test cache behavior
6. **Performance**: Test transformation performance
7. **Edge Cases**: Test edge cases
8. **Type Safety**: Test type safety

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HarshParmar25) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
