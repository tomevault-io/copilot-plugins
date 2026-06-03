## quickstart

> // Saleor Quickstart - Cursor AI Rules

// Saleor Quickstart - Cursor AI Rules
// This file provides instructions for Cursor AI to help with Saleor development

// Core technologies used in this project
const technologies = [
  "Next.js",
  "TypeScript",
  "GraphQL",
  "Saleor",
  "Tailwind CSS",
  "Apollo Client"
];

// Best practices for Saleor GraphQL development
const bestPractices = [
  "Use fragments for reusable parts of GraphQL queries",
  "Implement proper error handling and loading states for all GraphQL operations",
  "Follow the Configurator patterns for extending product types and attributes",
  "Use TypeScript types generated from the GraphQL schema",
  "Leverage Apollo Client's caching capabilities for performance",
  "Structure GraphQL operations into queries, mutations and fragments",
  "Implement dynamic form generation based on Saleor's attribute system",
  "Follow the multi-channel architecture pattern",
  "Handle checkout process according to Saleor's step-based approach",
  "Respect price freezing and stock allocation patterns for orders"
];

// Folder structure guide
const folderStructure = `
src/
  components/           # React components
    products/           # Product-related components
    checkout/           # Checkout flow components
    ui/                 # Shared UI components
  graphql/              # GraphQL operations
    fragments/          # Reusable query fragments
    queries/            # GraphQL queries
    mutations/          # GraphQL mutations
  hooks/                # Custom React hooks
  lib/                  # Utility functions
  pages/                # Next.js pages
  styles/               # Global styles
  providers/            # Context providers
  types/                # TypeScript types
`;

// Saleor Core Concepts
const saleorCoreConcepts = `
1. Multi-Channel Architecture
   - Channels represent sales channels (websites, mobile apps, POS systems)
   - Each channel has its own pricing, availability, and shipping options
   - Products can be available in multiple channels with different configurations

2. Pricing System
   - Supports gross and net pricing
   - Handles multiple currencies per channel
   - Implements price lists and discounts
   - Manages tax configurations and calculations

3. Order Management Flow
   - Order states: Unconfirmed → Unfulfilled → Partially Fulfilled → Fulfilled → Completed
   - Supports partial fulfillments and returns
   - Implements payment flow with transaction states
   - Handles refunds and cancellations

4. Meta Fields System
   - Allows storing additional data on most entities
   - Custom fields can be added to orders, users, products, etc.
   - Useful for integrations with external systems

5. Permission System
   - RBAC (Role-Based Access Control)
   - Staff users with assigned permissions
   - Token-based API authentication
   - Permission-based access to API operations

6. Transaction System
   - Supports asynchronous payment flow
   - Transaction states: Pending → Success/Failed
   - Allows for payment refunds and captures
   - Integration with external payment providers via Apps

7. App Framework
   - Extends core functionality with Apps
   - Apps can provide webhooks, payment methods, etc.
   - Apps authenticate with tokens
   - Custom dashboard extensions via Apps
`;

// Saleor-specific patterns
const saleorPatterns = `
1. Configurator-based Product Modeling
   - Define product types with appropriate attributes
   - Use variant attributes for options that create distinct variants
   - Use product attributes for features that don't create variants

2. Multi-Channel Setup
   - Always include channel in GraphQL operations
   - Handle channel-specific pricing and availability

3. Checkout Flow
   - Follow the step sequence: AddItems → ShippingAddress → ShippingMethod → Billing → Payment
   - Handle shipping and billing address validation
   - Implement proper error handling for checkout mutations

4. Dynamic Attribute Handling
   - Render form fields based on attribute input types
   - Use appropriate UI components for each attribute type
   - Generate variant combinations when needed

5. Authentication and Permissions
   - Use JWT tokens for authentication
   - Implement proper permission checks
`;

// GraphQL schema awareness - Extended with deeper schema examples
const schemaPatterns = `
1. Product and Variant Structure
   - Product is the parent entity with general information
   - ProductVariant represents sellable variants with unique SKUs
   - Products belong to ProductTypes which define their attributes
   - Example of fetching a product with variants:
   
   query ProductWithVariants($id: ID!, $channel: String!) {
     product(id: $id, channel: $channel) {
       id
       name
       slug
       description
       category { name, slug }
       collections { edges { node { name, slug } } }
       media { url, alt, type }
       variants {
         id
         name
         sku
         attributes {
           attribute { name, slug }
           values { name, slug }
         }
         pricing {
           price { gross { amount, currency } }
           onSale
           discount { gross { amount } }
         }
         quantityAvailable
       }
     }
   }

2. Order and Checkout Flow
   - Checkout is a temporary cart that becomes an Order
   - Orders have lines, fulfillments, payments, and transactions
   - Example of order creation flow:
   
   # 1. Create checkout
   mutation CheckoutCreate($input: CheckoutCreateInput!) {
     checkoutCreate(input: $input) {
       checkout { id, token }
       errors { field, message, code }
     }
   }
   
   # 2. Add checkout lines
   mutation CheckoutLinesAdd($checkoutId: ID!, $lines: [CheckoutLineInput!]!) {
     checkoutLinesAdd(checkoutId: $checkoutId, lines: $lines) {
       checkout { id, totalPrice { gross { amount, currency } } }
       errors { field, message, code }
     }
   }
   
   # 3. Add shipping address
   mutation CheckoutShippingAddressUpdate($checkoutId: ID!, $address: AddressInput!) {
     checkoutShippingAddressUpdate(checkoutId: $checkoutId, shippingAddress: $address) {
       checkout { id }
       errors { field, message, code }
     }
   }
   
   # 4. Select shipping method
   mutation CheckoutShippingMethodUpdate($checkoutId: ID!, $shippingMethodId: ID!) {
     checkoutShippingMethodUpdate(checkoutId: $checkoutId, shippingMethodId: $shippingMethodId) {
       checkout { id }
       errors { field, message, code }
     }
   }
   
   # 5. Complete checkout (creates order)
   mutation CheckoutComplete($checkoutId: ID!) {
     checkoutComplete(checkoutId: $checkoutId) {
       order { id, status, number }
       errors { field, message, code }
     }
   }

3. Transaction System
   - Transactions represent payment operations
   - Used for payment processing, captures, and refunds
   - Example of transaction operations:
   
   # Initialize a transaction
   mutation TransactionInitialize($id: ID!, $amount: PositiveDecimal!, $paymentGateway: PaymentGatewayToInitialize!) {
     transactionInitialize(id: $id, amount: $amount, paymentGateway: $paymentGateway) {
       transaction {
         id
         actions
         status
         createdAt
         modifiedAt
         authorizedAmount { amount, currency }
         chargedAmount { amount, currency }
         refundedAmount { amount, currency }
         canceledAmount { amount, currency }
         data
       }
       errors { field, message, code }
     }
   }
   
   # Process a transaction
   mutation TransactionProcess($id: ID!, $data: JSON) {
     transactionProcess(id: $id, data: $data) {
       transaction {
         id
         status
         # Other transaction fields
       }
       errors { field, message, code }
     }
   }
   
   # Request a refund
   mutation OrderRefund($id: ID!, $amount: PositiveDecimal!, $transactionId: ID) {
     orderRefund(id: $id, amount: $amount, transactionId: $transactionId) {
       order { id, status }
       errors { field, message, code }
     }
   }

4. App Framework
   - Apps extend Saleor's functionality
   - They can provide payment gateways, custom fields, and webhooks
   - Example of app installation and token creation:
   
   # Install app
   mutation AppInstall($input: AppInstallInput!) {
     appInstall(input: $input) {
       appInstallation {
         id
         status
         appName
       }
       errors { field, message, code }
     }
   }
   
   # Create app token
   mutation AppTokenCreate($input: AppTokenInput!) {
     appTokenCreate(input: $input) {
       appToken {
         id
         name
         lastUsedAt
       }
       authToken
       errors { field, message, code }
     }
   }

5. Webhook System
   - Webhooks allow apps to receive events from Saleor
   - Example of webhook creation:
   
   mutation WebhookCreate($input: WebhookCreateInput!) {
     webhookCreate(input: $input) {
       webhook {
         id
         name
         events
         targetUrl
         secretKey
       }
       errors { field, message, code }
     }
   }
`;

// Configurator-specific guidance
const configuratorGuidance = `
1. Data Model Configuration
   - Define and extend the schema using config.yml
   - The configurator allows you to setup:
     * Product Types and their attributes
     * Categories and collections
     * Page Types for content
     * Shipping and payment methods
     * Channels and currencies
   
   # Example of configuring a complex product type
   productTypes:
     - name: "Smartphone"
       hasVariants: true
       taxClass: "standard"
       weight: 0.5
       isShippingRequired: true
       isDigital: false
       productAttributes:
         - name: "Brand"
           inputType: "DROPDOWN"
           values:
             - name: "Apple"
             - name: "Samsung"
             - name: "Xiaomi"
         - name: "Warranty"
           inputType: "RICH_TEXT"
         - name: "Manual"
           inputType: "FILE"
       variantAttributes:
         - name: "Storage"
           inputType: "DROPDOWN"
           values:
             - name: "64GB"
             - name: "128GB"
             - name: "256GB"
         - name: "Color"
           inputType: "SWATCH"
           values:
             - name: "Black"
             - name: "White"
             - name: "Blue"

2. Shop Configuration
   - Configure global shop settings
   - Example of shop configuration:
   
   shop:
     name: "Saleor Demo Store"
     description: "Modern e-commerce platform"
     customerAllowedToSetExternalReference: true
     defaultMailSenderName: "Saleor Team"
     defaultMailSenderAddress: "info@example.com"
     limitQuantityPerCheckout: 50
     displayGrossPrices: true
     chargeTaxesOnShipping: true
     includeTaxesInPrices: true
     allowOrder: true
     reserveStockDurationAnonymousUser: 60
     reserveStockDurationAuthenticatedUser: 120

3. Channel Configuration
   - Setup sales channels with their currencies and countries
   - Example of channel configuration:
   
   channels:
     - name: "Web Store"
       slug: "web-store"
       currencyCode: "USD"
       defaultCountry: "US"
       supportedCountries:
         - "US"
         - "CA"
       defaultLocale: "en-US"
       stockSettings:
         trackInventory: true
         reserveStock: true
       orderSettings:
         automaticallyConfirmAllNewOrders: true
         automaticallyFulfillNonShippableGiftCard: true
         expireOrdersAfter: 30
         allowUnpaidOrders: true

4. Data Loading Patterns
   - Configure data loading from external sources
   - Example product import configuration:
   
   productImport:
     - source: "csv"
       filePath: "./products.csv"
       batchSize: 100
       mapping:
         nameColumn: "Product Name"
         descriptionColumn: "Description"
         skuColumn: "SKU"
         priceColumn: "Price"
         categoryColumn: "Category"
         imageColumns: ["Image 1", "Image 2"]
         attributeColumns:
           "Brand": "Brand"
           "Color": "Color"
           "Size": "Size"
`;

// Saleor Storefront guidance (replacing the component/frontend configurator guidance)
const storefrontGuidance = {
  // 1. Product listing patterns
  productListingPattern: `
import { useProductList } from '@/hooks/useProductList';

export const ProductGrid = ({ categoryId, first = 20 }) => {
  const { products, loading, error, fetchMore } = useProductList({ 
    categoryId, 
    first 
  });
  
  if (loading) return <ProductGridSkeleton />;
  if (error) return <ErrorDisplay message={error} />;
  
  return (
    <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
      {products.pageInfo.hasNextPage && (
        <button 
          className="btn-load-more" 
          onClick={() => fetchMore({
            variables: { after: products.pageInfo.endCursor }
          })}
        >
          Load more
        </button>
      )}
    </div>
  );
}`,

  // 2. Product detail pattern
  productDetailPattern: `
import { useProduct } from '@/hooks/useProduct';
import { useAddToCart } from '@/hooks/useCart';

export const ProductDetail = ({ id, channelSlug = 'default-channel' }) => {
  const { product, loading, error } = useProduct(id, channelSlug);
  const { addToCart, isAdding } = useAddToCart();
  const [selectedVariantId, setSelectedVariantId] = useState(null);
  
  if (loading) return <ProductDetailSkeleton />;
  if (error) return <ErrorDisplay message={error} />;
  if (!product) return <ProductNotFound />;
  
  // Handle variant selection based on selected attributes
  const handleAddToCart = () => {
    if (!selectedVariantId) return;
    
    addToCart({
      variantId: selectedVariantId,
      quantity: 1
    });
  };
  
  return (
    <div className="product-detail">
      <div className="product-gallery">
        {product.media.map(media => (
          <Image 
            key={media.id}
            src={media.url}
            alt={media.alt}
            width={600}
            height={600}
          />
        ))}
      </div>
      
      <div className="product-info">
        <h1 className="product-name">{product.name}</h1>
        <div className="product-price">
          {formatPrice(product.pricing.priceRange.start.gross)}
        </div>
        
        <div className="product-description">
          {product.description}
        </div>
        
        {product.variants.length > 1 && (
          <VariantSelector 
            product={product}
            selectedVariantId={selectedVariantId}
            onVariantChange={setSelectedVariantId}
          />
        )}
        
        <button 
          className="btn-add-to-cart"
          disabled={isAdding || !selectedVariantId}
          onClick={handleAddToCart}
        >
          {isAdding ? 'Adding...' : 'Add to Cart'}
        </button>
      </div>
    </div>
  );
}`,

  // 3. Checkout flow pattern
  checkoutFlowPattern: `
import { useCheckout } from '@/hooks/useCheckout';

export const CheckoutPage = () => {
  const { 
    checkout, 
    loading, 
    error,
    addShippingAddress,
    selectShippingMethod,
    addBillingAddress,
    completeCheckout
  } = useCheckout();
  
  const [step, setStep] = useState('address'); // address, shipping, payment
  
  // Handle form submissions for each step
  const handleAddressSubmit = async (addressData) => {
    await addShippingAddress(addressData);
    setStep('shipping');
  };
  
  const handleShippingMethodSubmit = async (methodId) => {
    await selectShippingMethod(methodId);
    setStep('payment');
  };
  
  const handlePaymentSubmit = async (paymentData) => {
    // Initialize payment transaction
    // Then complete checkout
    const result = await completeCheckout();
    
    if (result.order) {
      // Redirect to order confirmation
      router.push(\`/order/\${result.order.id}\`);
    }
  };
  
  return (
    <div className="checkout-page">
      {/* Checkout steps */}
      <CheckoutSteps currentStep={step} />
      
      {/* Checkout forms based on current step */}
      {step === 'address' && (
        <AddressForm onSubmit={handleAddressSubmit} />
      )}
      
      {step === 'shipping' && (
        <ShippingMethodSelector 
          methods={checkout.availableShippingMethods}
          onSubmit={handleShippingMethodSubmit} 
        />
      )}
      
      {step === 'payment' && (
        <PaymentForm 
          totalAmount={checkout.totalPrice.gross.amount}
          currency={checkout.totalPrice.gross.currency}
          onSubmit={handlePaymentSubmit}
        />
      )}
      
      {/* Order summary */}
      <OrderSummary checkout={checkout} />
    </div>
  );
}`,

  // 4. Multi-channel support pattern
  multiChannelPattern: `
import { useChannelContext } from '@/contexts/ChannelContext';

// Channel context provider
export const ChannelProvider = ({ children }) => {
  const [activeChannel, setActiveChannel] = useState({
    slug: 'default-channel',
    currencyCode: 'USD'
  });
  
  // Load available channels from API
  const { data } = useQuery(GET_CHANNELS);
  const availableChannels = data?.channels || [];
  
  const changeChannel = (channelSlug) => {
    const channel = availableChannels.find(ch => ch.slug === channelSlug);
    if (channel) {
      setActiveChannel(channel);
      // Update localStorage for persistence
      localStorage.setItem('activeChannel', JSON.stringify(channel));
    }
  };
  
  return (
    <ChannelContext.Provider value={{ 
      activeChannel, 
      availableChannels,
      changeChannel 
    }}>
      {children}
    </ChannelContext.Provider>
  );
};

// Usage in components
export const ChannelSwitcher = () => {
  const { activeChannel, availableChannels, changeChannel } = useChannelContext();
  
  return (
    <select 
      value={activeChannel.slug}
      onChange={(e) => changeChannel(e.target.value)}
      className="channel-switcher"
    >
      {availableChannels.map(channel => (
        <option key={channel.slug} value={channel.slug}>
          {channel.name} ({channel.currencyCode})
        </option>
      ))}
    </select>
  );
};`,
};

// Saleor App framework and dummy payment patterns (replacing TypeScript guidelines)
const saleorAppPatterns = `
1. App Structure
   - Saleor Apps are standalone applications that extend Saleor
   - They can be installed via Dashboard or API
   - Structure of a typical app:
     * API handlers for webhooks and extensions
     * Dashboard views for configuration
     * Business logic for integrations

2. App Manifest
   - Defines app metadata, permissions, and extensions
   - Example app manifest:
   
   {
     "name": "Dummy Payment App",
     "version": "1.0.0",
     "about": "Provides dummy payment method for testing",
     "dataPrivacy": "Stores payment details for processing",
     "dataPrivacyUrl": "https://example.com/privacy",
     "homepageUrl": "https://example.com",
     "supportUrl": "https://example.com/support",
     "permissions": [
       "MANAGE_ORDERS",
       "MANAGE_PAYMENTS"
     ],
     "id": "saleor.app.dummy-payment",
     "appUrl": "https://example.com/app",
     "configurationUrl": "https://example.com/app/configuration",
     "tokenTargetUrl": "https://example.com/api/register"
   }

3. App Authentication
   - Apps authenticate with Saleor using JWT tokens
   - Example of verifying app tokens:
   
   import { AuthData, verifyJWT } from '@saleor/app-sdk/verify-jwt';
   
   export async function verifyToken(authToken: string, appId: string, saleorApiUrl: string): Promise<AuthData> {
     try {
       const verified = await verifyJWT({
         token: authToken,
         appId,
         saleorApiUrl,
       });
       
       return verified;
     } catch (error) {
       throw new Error('Invalid token');
     }
   }

4. Webhook Handling
   - Apps can subscribe to Saleor events via webhooks
   - Example of a webhook handler:
   
   // Next.js API route
   import { createWebhookHandler } from '@saleor/app-sdk/handlers/next';
   
   export const webhookHandler = createWebhookHandler({
     async webhookHandler(req, res, ctx) {
       const { event, payload } = ctx;
       
       switch (event) {
         case 'ORDER_CREATED':
           // Handle order creation
           await processNewOrder(payload);
           break;
         
         case 'PAYMENT_AUTHORIZE':
           // Handle payment authorization
           await authorizePayment(payload);
           break;
           
         // Handle other events
       }
       
       return res.status(200).json({ success: true });
     }
   });
   
   export default webhookHandler;

5. Dummy Payment Implementation
   - Example of a dummy payment provider:
   
   // Dummy payment handler
   export async function dummyPaymentHandler(req, res) {
     const { amount, currency, checkoutId, returnUrl } = req.body;
     
     // Generate unique transaction ID
     const transactionId = \`dummy-\${Date.now()}-\${Math.random().toString(36).substring(2, 10)}\`;
     
     // Store transaction details in your database
     await storeTransaction({
       id: transactionId,
       amount,
       currency,
       checkoutId,
       status: 'PENDING'
     });
     
     // In a real implementation, redirect to payment provider
     // For dummy payment, simulate success after delay
     setTimeout(async () => {
       // Update transaction to success
       await updateTransaction(transactionId, { status: 'SUCCESS' });
       
       // Call back to Saleor API to update the payment
       await notifySaleorAboutSuccessfulPayment(checkoutId, transactionId, amount, currency);
     }, 5000);
     
     // Return redirect URL for customer
     res.status(200).json({
       redirectUrl: \`\${returnUrl}?transaction=\${transactionId}\`,
       transactionId
     });
   }

6. App Extension Points
   - Dashboard extensions
   - Payment gateways
   - Shipping methods
   - Example of registering a payment app:
   
   mutation PaymentGatewayInitialize($id: ID!) {
     paymentGatewayInitialize(id: $id, paymentGateways: [
       {
         id: "dummy-payment",
         name: "Dummy Payment",
         config: [
           { field: "dummyTransactionTime", value: "5" },
           { field: "autoConfirm", value: "true" }
         ]
       }
     ]) {
       errors {
         field
         message
       }
     }
   }
`;

// Transaction system implementation patterns
const transactionPatterns = `
1. Transaction Flow
   - Transactions represent payment operations in Saleor
   - Flow: Initialize → Process → Complete/Fail
   - Transactions can be authorized, charged, refunded, or canceled
   - Example of a complete transaction flow:

2. Transaction Initialization
   - Start a payment transaction for checkout or order
   - Example GraphQL mutation:
   
   mutation TransactionInitialize($id: ID!, $amount: PositiveDecimal!, $paymentGateway: PaymentGatewayToInitialize!) {
     transactionInitialize(id: $id, amount: $amount, paymentGateway: $paymentGateway) {
       transaction {
         id
         createdAt
         actions    # Available actions: CHARGE, REFUND, CANCEL
         status
         type
         data      # Custom JSON data from payment provider
         amount {
           amount
           currency
         }
       }
       errors {
         field
         message
         code
       }
     }
   }

3. Transaction Processing
   - Process payment with data from payment provider
   - Example GraphQL mutation:
   
   mutation TransactionProcess($id: ID!, $data: JSON, $amount: PositiveDecimal) {
     transactionProcess(id: $id, data: $data, amount: $amount) {
       transaction {
         id
         status
         modifiedAt
         actions
         events {    # Transaction events history
           id
           amount {
             amount
             currency
           }
           type      # AUTHORIZATION, CHARGE, CANCEL, etc.
           createdAt
           status    # SUCCESS, FAILURE, PENDING
           reference # Reference from payment provider
         }
       }
       errors {
         field
         message
         code
       }
     }
   }

4. Transaction Actions
   - Perform actions on transactions like charging or refunding
   - Example refund mutation:
   
   mutation TransactionRefund($id: ID!, $amount: PositiveDecimal!) {
     transactionRefund(id: $id, amount: $amount) {
       transaction {
         id
         status
         refundedAmount {
           amount
           currency
         }
       }
       errors {
         field
         message
         code
       }
     }
   }

5. Transaction Events
   - Track history of transaction operations
   - Query transaction events:
   
   query TransactionEvents($id: ID!) {
     transaction(id: $id) {
       events {
         id
         createdAt
         status
         type
         amount {
           amount
           currency
         }
         reference
         message
       }
     }
   }

6. Implementing Transaction in a Payment App
   - Flow for implementing a payment gateway in an app:
     1. App receives webhook with TRANSACTION_INITIALIZE event
     2. App creates payment in external provider
     3. App returns payment URL or token
     4. Customer completes payment
     5. App receives confirmation from provider
     6. App calls TRANSACTION_PROCESS mutation to update Saleor

   // Example webhook handler for transaction initialize
   async function handleTransactionInitialize(payload) {
     const { 
       transaction: { id: transactionId, amount, currency },
       order,
       action: { actionType, redirectUrl }
     } = payload;
     
     // Create payment in external provider
     const paymentResponse = await externalPaymentProvider.createPayment({
       amount: amount.amount,
       currency: amount.currency,
       orderId: order.id,
       customerEmail: order.userEmail,
       returnUrl: redirectUrl,
       // Add other required fields
     });
     
     // Store payment details in your database
     await storePaymentDetails({
       transactionId,
       paymentId: paymentResponse.id,
       status: 'PENDING'
     });
     
     // Return data to be saved with the transaction
     return {
       data: JSON.stringify({
         paymentId: paymentResponse.id,
         paymentUrl: paymentResponse.checkoutUrl
       }),
       pspReference: paymentResponse.id,
       result: 'PENDING',
     };
   }
   
   // Example webhook handler for transaction process
   async function handleTransactionProcess(payload) {
     const { transactionId, data } = payload;
     
     // Parse transaction data
     const parsedData = JSON.parse(data);
     const { paymentId } = parsedData;
     
     // Check payment status in external provider
     const paymentStatus = await externalPaymentProvider.getPaymentStatus(paymentId);
     
     if (paymentStatus === 'COMPLETED') {
       // Payment was successful
       // Call Saleor API to update transaction
       await saleorClient.mutate({
         mutation: TRANSACTION_PROCESS_MUTATION,
         variables: {
           id: transactionId,
           data: JSON.stringify({ status: 'COMPLETED', reference: paymentId }),
           amount: amount
         }
       });
     } else if (paymentStatus === 'FAILED') {
       // Payment failed
       // Call Saleor API to mark transaction as failed
       await saleorClient.mutate({
         mutation: TRANSACTION_PROCESS_MUTATION,
         variables: {
           id: transactionId,
           data: JSON.stringify({ 
             status: 'FAILED', 
             reference: paymentId,
             errorMessage: 'Payment processing failed'
           })
         }
       });
     }
   }
`;

// Additional instructions
const additionalInstructions = `
1. Use Apollo Provider at the root of your app for GraphQL client setup

2. Create custom hooks for common GraphQL operations:
   - useProduct(id)
   - useProductList(params)
   - useCategory(id)
   - useSearch(query)
   - useCheckout()

3. Implement proper loading and error states for all GraphQL operations

4. Use TypeScript for type safety with GraphQL operations:
   - Import types generated from the GraphQL schema
   - Define prop interfaces for all components
   - Use nullable types appropriately

5. Follow Saleor naming conventions for all variables, functions, and components

6. Generate product variants using the cartesian product of attribute values

7. Implement dynamic form generation for product types and attributes

8. Handle multi-currency and multi-language support properly
`;

// Common GraphQL operations pattern
const graphqlPatterns = {
  productQuery: `
query ProductDetails($id: ID!, $channel: String!) {
  product(id: $id, channel: $channel) {
    id
    name
    description
    # Include other fields as needed
    pricing {
      priceRange {
        start {
          gross {
            amount
            currency
          }
        }
      }
    }
    # Use fragments for variant data
    ...ProductVariantsFragment
  }
}`,

  checkoutCreate: `
mutation CheckoutCreate($input: CheckoutCreateInput!) {
  checkoutCreate(input: $input) {
    checkout {
      id
      token
      # Include other required fields
    }
    errors {
      field
      message
      code
    }
  }
}`
};

// Configurator implementation patterns - Added guide for generating code
const configuratorImplementationPatterns = {
  // 1. Config.yml patterns
  configYaml: `
# Product Type with variants
productTypes:
  - name: "T-Shirt"
    hasVariants: true
    attributes:
      - name: "Size"
        inputType: "DROPDOWN"
        values:
          - name: "Small"
          - name: "Medium"
          - name: "Large"
      - name: "Color"
        inputType: "SWATCH"
        values:
          - name: "Red"
          - name: "Blue"
          - name: "Green"

# Content/Page Type
pageTypes:
  - name: "Blog Post"
    attributes:
      - name: "Author"
        inputType: "PLAIN_TEXT"
      - name: "PublishDate"
        inputType: "DATE"
      - name: "Categories"
        inputType: "MULTISELECT"
        values:
          - name: "News"
          - name: "Tutorial"
  `,

  // 2. GraphQL queries for products with attributes
  productWithAttributesQuery: `
query ProductWithAttributes($id: ID!, $channel: String!) {
  product(id: $id, channel: $channel) {
    id
    name
    description
    # Media and SEO fields
    thumbnail { url, alt }
    media { url, alt }
    seoTitle
    seoDescription
    
    # Pricing information
    pricing {
      priceRange {
        start {
          gross { amount, currency }
          net { amount, currency }
        }
      }
      discount { gross { amount, currency } }
    }
    
    # Product attributes
    attributes {
      attribute {
        id
        name
        slug
        inputType
      }
      values {
        id
        name
        # For rich text content
        richText
        # For file attributes
        file { url, contentType }
      }
    }
    
    # Variants and their attributes
    variants {
      id
      name
      sku
      quantityAvailable
      pricing {
        price { gross { amount, currency } }
        discount { gross { amount, currency } }
      }
      attributes {
        attribute {
          id
          name
          slug
        }
        values {
          id
          name
        }
      }
    }
  }
}`,

  // 3. GraphQL query for pages with attributes
  pageWithAttributesQuery: `
query PageWithAttributes($id: ID!) {
  page(id: $id) {
    id
    title
    slug
    content
    # SEO fields
    seoTitle
    seoDescription
    
    # Page attributes
    attributes {
      attribute {
        id
        name
        slug
        inputType
      }
      values {
        id
        name
        # For rich text content
        richText
        # For file attributes
        file { url, contentType }
      }
    }
  }
}`,

  // 4. Hook for fetching products with attributes
  useProductWithAttributesHook: `
import { useQuery } from '@apollo/client';
import { 
  ProductWithAttributesDocument, 
  type ProductWithAttributesQuery,
  type ProductWithAttributesQueryVariables
} from '../generated/graphql';

export function useProductWithAttributes(id: string, channel: string) {
  const { data, loading, error } = useQuery<
    ProductWithAttributesQuery,
    ProductWithAttributesQueryVariables
  >(
    ProductWithAttributesDocument,
    {
      variables: { id, channel },
      skip: !id || !channel,
      // Cache for 5 minutes
      fetchPolicy: 'cache-and-network',
    }
  );

  return {
    product: data?.product,
    loading,
    error: error ? error.message : null
  };
}`,

  // 5. Attribute renderer component for different attribute types
  attributeRendererComponent: `
import React from 'react';
import { AttributeValue, AttributeInputTypeEnum } from 'types/generated/graphql';

interface AttributeRendererProps {
  attributeName: string;
  attributeType: AttributeInputTypeEnum;
  values: AttributeValue[];
}

export const AttributeRenderer: React.FC<AttributeRendererProps> = ({
  attributeName,
  attributeType,
  values
}) => {
  if (!values?.length) return null;
  
  switch (attributeType) {
    case AttributeInputTypeEnum.DROPDOWN:
    case AttributeInputTypeEnum.SWATCH:
      return (
        <div className="attribute">
          <span className="attribute-name">{attributeName}:</span>
          <span className="attribute-value">{values[0]?.name}</span>
        </div>
      );
    
    case AttributeInputTypeEnum.MULTISELECT:
      return (
        <div className="attribute">
          <span className="attribute-name">{attributeName}:</span>
          <div className="attribute-values">
            {values.map(value => (
              <span key={value.id} className="attribute-value-chip">
                {value.name}
              </span>
            ))}
          </div>
        </div>
      );
  
    case AttributeInputTypeEnum.RICH_TEXT:
      return (
        <div className="attribute">
          <span className="attribute-name">{attributeName}:</span>
          <div 
            className="attribute-rich-text"
            dangerouslySetInnerHTML={{ __html: values[0]?.richText ?? '' }} 
          />
        </div>
      );
      
    case AttributeInputTypeEnum.FILE:
      return (
        <div className="attribute">
          <span className="attribute-name">{attributeName}:</span>
          <a 
            href={values[0]?.file?.url ?? '#'} 
            target="_blank" 
            rel="noopener noreferrer"
            className="attribute-file-link"
          >
            {values[0]?.name}
          </a>
        </div>
      );
      
    case AttributeInputTypeEnum.REFERENCE:
      return (
        <div className="attribute">
          <span className="attribute-name">{attributeName}:</span>
          <span className="attribute-reference">
            {values[0]?.name} (ID: {values[0]?.id})
          </span>
        </div>
      );
    
    // Handle other attribute types
    default:
      return (
        <div className="attribute">
          <span className="attribute-name">{attributeName}:</span>
          <span className="attribute-value">{values[0]?.name}</span>
        </div>
      );
  }
}`,

  // 6. Dynamic form component for product attribute input
  attributeFormComponent: `
import React from 'react';
import { 
  Attribute, 
  AttributeValue, 
  AttributeInputTypeEnum 
} from 'types/generated/graphql';

interface AttributeFormProps {
  attribute: Attribute;
  selectedValues: AttributeValue[];
  onChange: (attributeId: string, valueIds: string[]) => void;
}

export const AttributeForm: React.FC<AttributeFormProps> = ({
  attribute,
  selectedValues,
  onChange
}) => {
  const handleChange = (event: React.ChangeEvent<HTMLSelectElement | HTMLInputElement>) => {
    const valueId = event.target.value;
    
    if (attribute.inputType === AttributeInputTypeEnum.MULTISELECT) {
      const isChecked = (event.target as HTMLInputElement).checked;
      const currentValues = selectedValues.map(v => v.id);
      
      if (isChecked) {
        onChange(attribute.id, [...currentValues, valueId]);
      } else {
        onChange(attribute.id, currentValues.filter(id => id !== valueId));
      }
    } else {
      onChange(attribute.id, [valueId]);
    }
  };
  
  const renderAttributeInput = () => {
    const choices = attribute.choices?.edges.map(edge => edge.node) || [];
    
    switch (attribute.inputType) {
      case AttributeInputTypeEnum.DROPDOWN:
        return (
          <select
            className="form-select"
            value={selectedValues[0]?.id || ''}
            onChange={handleChange}
          >
            <option value="">Select {attribute.name}</option>
            {choices.map(choice => (
              <option key={choice.id} value={choice.id}>
                {choice.name}
              </option>
            ))}
          </select>
        );
        
      case AttributeInputTypeEnum.MULTISELECT:
        return (
          <div className="checkbox-group">
            {choices.map(choice => {
              const isSelected = selectedValues.some(v => v.id === choice.id);
              return (
                <label key={choice.id} className="checkbox-label">
                  <input
                    type="checkbox"
                    value={choice.id}
                    checked={isSelected}
                    onChange={handleChange}
                    className="checkbox-input"
                  />
                  {choice.name}
                </label>
              );
            })}
          </div>
        );
      
      case AttributeInputTypeEnum.SWATCH:
        return (
          <div className="swatch-group">
            {choices.map(choice => {
              const isSelected = selectedValues.some(v => v.id === choice.id);
              return (
                <button
                  key={choice.id}
                  type="button"
                  className={\`swatch \${isSelected ? 'selected' : ''}\`}
                  style={{ backgroundColor: choice.name.toLowerCase() }}
                  onClick={() => onChange(attribute.id, [choice.id])}
                  aria-label={choice.name}
                />
              );
            })}
          </div>
        );
      
      // Add more input types as needed
      
      default:
        return (
          <input
            type="text"
            className="form-input"
            value={selectedValues[0]?.name || ''}
            onChange={(e) => {
              // This would require a custom handling for plain text attributes
              // as they might not have predefined values
            }}
            placeholder={\`Enter \${attribute.name}\`}
          />
        );
    }
  };
  
  return (
    <div className="attribute-form-field">
      <label className="attribute-label">{attribute.name}</label>
      {renderAttributeInput()}
    </div>
  );
}`,

  // 7. Product variant selector component
  variantSelectorComponent: `
import React, { useMemo } from 'react';
import { Product, ProductVariant } from 'types/generated/graphql';

interface VariantSelectorProps {
  product: Product;
  selectedVariantId: string | null;
  onVariantChange: (variantId: string) => void;
}

export const VariantSelector: React.FC<VariantSelectorProps> = ({
  product,
  selectedVariantId,
  onVariantChange
}) => {
  // Group variants by attribute values
  const variantAttributes = useMemo(() => {
    const variantAttributeMap = new Map();
    
    // Find variant attributes
    product.productType?.variantAttributes?.forEach(attr => {
      if (attr) {
        variantAttributeMap.set(attr.id, {
          id: attr.id,
          name: attr.name,
          values: new Set()
        });
      }
    });
    
    // Collect all possible values for each variant attribute
    product.variants?.forEach(variant => {
      variant?.attributes?.forEach(attrValue => {
        const attrId = attrValue.attribute.id;
        const attrData = variantAttributeMap.get(attrId);
        
        if (attrData) {
          attrValue.values.forEach(val => {
            attrData.values.add(val);
          });
        }
      });
    });
    
    return Array.from(variantAttributeMap.values());
  }, [product]);
  
  // Find available combinations based on current selection
  const [selectedAttributes, setSelectedAttributes] = useState<Record<string, string>>({});
  
  // Determine which variant matches the selected attributes
  const matchingVariant = useMemo(() => {
    if (Object.keys(selectedAttributes).length === 0) {
      return product.variants?.[0] || null;
    }
    
    return product.variants?.find(variant => {
      return variant?.attributes?.every(attrValue => {
        const selectedValueId = selectedAttributes[attrValue.attribute.id];
        if (!selectedValueId) return true;
        
        return attrValue.values.some(val => val.id === selectedValueId);
      });
    }) || null;
  }, [product.variants, selectedAttributes]);
  
  // Update selected variant when attributes change
  useEffect(() => {
    if (matchingVariant?.id) {
      onVariantChange(matchingVariant.id);
    }
  }, [matchingVariant, onVariantChange]);
  
  const handleAttributeChange = (attributeId: string, valueId: string) => {
    setSelectedAttributes(prev => ({
      ...prev,
      [attributeId]: valueId
    }));
  };
  
  return (
    <div className="variant-selector">
      {variantAttributes.map(attr => (
        <div key={attr.id} className="variant-attribute">
          <span className="variant-attribute-name">{attr.name}:</span>
          <div className="variant-attribute-values">
            {Array.from(attr.values).map(value => (
              <button
                key={value.id}
                className={\`variant-value \${selectedAttributes[attr.id] === value.id ? 'selected' : ''}\`}
                onClick={() => handleAttributeChange(attr.id, value.id)}
              >
                {value.name}
              </button>
            ))}
          </div>
        </div>
      ))}
      
      {matchingVariant && (
        <div className="variant-details">
          <div className="variant-price">
            {matchingVariant.pricing?.price?.gross.amount} 
            {matchingVariant.pricing?.price?.gross.currency}
          </div>
          <div className="variant-availability">
            {matchingVariant.quantityAvailable > 0 
              ? \`In stock: \${matchingVariant.quantityAvailable}\` 
              : 'Out of stock'}
          </div>
        </div>
      )}
    </div>
  );
}`,

  // 8. Payment app integration for checkout
  paymentIntegrationCode: `
import { useState } from 'react';
import { useMutation } from '@apollo/client';
import { 
  TransactionInitializeDocument,
  TransactionInitializeMutation,
  TransactionInitializeMutationVariables
} from 'types/generated/graphql';

export function usePaymentInitialization() {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  const [initializeTransaction] = useMutation<
    TransactionInitializeMutation,
    TransactionInitializeMutationVariables
  >(TransactionInitializeDocument);
  
  const initializePayment = async ({
    checkoutId,
    amount,
    appId = 'dummy-payment-app',
    redirectUrl = window.location.origin + '/checkout/payment-confirmation'
  }) => {
    setIsLoading(true);
    setError(null);
    
    try {
      const { data } = await initializeTransaction({
        variables: {
          id: checkoutId,
          amount: amount,
          paymentGateway: {
            id: appId,
            data: JSON.stringify({
              redirectUrl,
              // Additional app-specific data
            })
          }
        }
      });
      
      const result = data?.transactionInitialize;
      
      if (result?.errors?.length) {
        throw new Error(result.errors[0].message);
      }
      
      // Get transaction data from response
      const transactionData = result?.transaction?.data;
      const transactionId = result?.transaction?.id;
      
      // Parse app response from transaction data
      const parsedData = transactionData 
        ? JSON.parse(transactionData)
        : null;
      
      return {
        transactionId,
        ...parsedData
      };
    } catch (err) {
      const errorMessage = err instanceof Error 
        ? err.message 
        : 'Payment initialization failed';
      setError(errorMessage);
      throw err;
    } finally {
      setIsLoading(false);
    }
  };
  
  return {
    initializePayment,
    isLoading,
    error
  };
}`
};

// Custom component patterns based on the Saleor schema
const componentPatterns = {
  productCard: `
// @cursor-component ProductCard
// @cursor-description Displays product card in listing pages
interface ProductCardProps {
  product: Product;
  layout?: "grid" | "list";
}

export const ProductCard: React.FC<ProductCardProps> = ({ 
  product, 
  layout = "grid" 
}) => {
  const { name, thumbnail, pricing } = product;
  
  return (
    <div className="product-card">
      {/* Tailwind classes based on layout */}
      <div className={\`\${layout === "grid" ? "flex-col" : "flex-row"} flex\`}>
        {/* Image component with proper loading */}
        <Image 
          src={thumbnail?.url || "/placeholder.jpg"} 
          alt={name}
          width={300}
          height={300}
          className="product-image"
        />
        <div className="product-info">
          <h3 className="product-name">{name}</h3>
          <div className="product-price">
            {formatPrice(pricing?.priceRange?.start?.gross)}
          </div>
          {/* Add to cart button or other actions */}
        </div>
      </div>
    </div>
  );
};`,

  attributeField: `
// @cursor-component AttributeField
// @cursor-description Renders a form field based on attribute type
interface AttributeFieldProps {
  attribute: Attribute;
  value: any;
  onChange: (value: any) => void;
  error?: string;
}

export const AttributeField: React.FC<AttributeFieldProps> = ({
  attribute,
  value,
  onChange,
  error
}) => {
  // Render different input types based on attribute.inputType
  switch (attribute.inputType) {
    case AttributeInputTypeEnum.DROPDOWN:
      return (
        <select
          value={value}
          onChange={(e) => onChange(e.target.value)}
          className="form-select"
        >
          <option value="">Select {attribute.name}</option>
          {attribute.values?.map((val) => (
            <option key={val.id} value={val.id}>
              {val.name}
            </option>
          ))}
        </select>
      );
    // Handle other input types...
    default:
      return (
        <input
          type="text"
          value={value}
          onChange={(e) => onChange(e.target.value)}
          className="form-input"
          placeholder={attribute.name}
        />
      );
  }
};`
}; 

---
> Source: [saleor/quickstart](https://github.com/saleor/quickstart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
