## sunu-rekolt

> Start --> Auth[Login / Signup]

flowchart TD
    Start[Start]
    Start --> Auth[Login / Signup]
    Auth --> RoleChoice[Select Role]
    RoleChoice --> Farmer[Agriculteur]
    RoleChoice --> Buyer[Acheteur]
    RoleChoice --> Admin[Admin Panel]

    %% Farmer flow
    Farmer --> FDashboard[Farmer Dashboard]
    FDashboard --> FAddProduct[Add Product]
    FAddProduct --> FSubmit[Submit for Validation]
    FSubmit --> FPending[Status: Pending]
    FPending --> FApproved[Admin Approves]

    FDashboard --> FMyProducts[View My Products]
    FDashboard --> FOrders[My Orders]
    FOrders --> ShowBuyerPhone[Display Buyer Phone]
    FOrders --> FMarkDelivery[Mark as Delivered]
    FDashboard --> FCatalogue[Browse Catalogue]
    FCatalogue --> FBuyProduct[Buy Product]
    FDashboard --> FIntrants[Boutique Intrants]

    %% Buyer flow
    Buyer --> BCatalogue[Browse Catalogue]
    BCatalogue --> BAddCart[Add to Cart]
    BAddCart --> BCheckout[Checkout + PayDunya]
    BCheckout --> BConfirm[Order Placed]
    BConfirm --> BOrders[Order History]
    BOrders --> ShowFarmerPhone[Display Farmer Phone]
    BOrders --> BReceive[Confirm Reception]
    

    %% Farmer call back
    FOrders --> ShowBuyerPhone[Access Buyer Phone (SMS Notification)]

    %% Admin flow
    Admin --> ALogin[Admin Login]
    ALogin --> ADashboard[Admin Dashboard]
    ADashboard --> AValidate[Validate Products]
    ADashboard --> AUsers[Manage Users]
    ADashboard --> AStats[View Stats]

    ADashboard --> AReports[Reports & Flags]

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LordThiouk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LordThiouk)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
