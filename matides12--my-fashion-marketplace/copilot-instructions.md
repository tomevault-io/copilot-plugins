## my-fashion-marketplace

> “master instruction” set up a Next.js (TypeScript) + Supabase project. It includes steps to create a Next.js app, set up Supabase, configure authentication, create the core database schema, and build initial pages (login, sign up, dashboard, product listing, etc.).


# Your rule content

─────────────────────────────────────────────────────────
MASTER INSTRUCTION FOR CURSOR AI
─────────────────────────────────────────────────────────
1. CREATE A NEW NEXT.JS PROJECT WITH TYPESCRIPT  
   - Use the following command (or equivalent in Cursor AI) to initialize the project:  
     npx create-next-app@latest my-fashion-marketplace --ts

2. SET UP SUPABASE  
   a) Install the Supabase client:  
      npm install @supabase/supabase-js  
   b) Create a Supabase account (if not already done) at supabase.com, then set up a new project.  
   c) In the Supabase dashboard, create the following tables (users, products, product_images, orders, comments, likes) using SQL or the table editor:

      --------------------------------------------------
      CREATE TABLE IF NOT EXISTS users (
        id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
        full_name   text,
        email       text UNIQUE NOT NULL,
        -- password is handled by Supabase Auth, but you can store additional data here
        role        text CHECK (role IN ('owner', 'customer')) DEFAULT 'customer',
        created_at  timestamptz DEFAULT now()
      );

      CREATE TABLE IF NOT EXISTS products (
        id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
        owner_id    uuid REFERENCES users(id) ON DELETE CASCADE,
        title       text NOT NULL,
        description text,
        price       numeric(10,2),
        is_active   boolean DEFAULT true,
        created_at  timestamptz DEFAULT now(),
        updated_at  timestamptz DEFAULT now()
      );

      CREATE TABLE IF NOT EXISTS product_images (
        id              uuid PRIMARY KEY DEFAULT gen_random_uuid(),
        product_id      uuid REFERENCES products(id) ON DELETE CASCADE,
        image_url       text NOT NULL,
        is_model_picture boolean DEFAULT false
      );

      CREATE TABLE IF NOT EXISTS orders (
        id            uuid PRIMARY KEY DEFAULT gen_random_uuid(),
        user_id       uuid REFERENCES users(id) ON DELETE CASCADE,
        product_id    uuid REFERENCES products(id) ON DELETE CASCADE,
        quantity      int DEFAULT 1,
        total_price   numeric(10,2),
        order_status  text CHECK (order_status IN ('pending','confirmed','shipped','delivered','cancelled'))
                     DEFAULT 'pending',
        created_at    timestamptz DEFAULT now(),
        updated_at    timestamptz DEFAULT now()
      );

      CREATE TABLE IF NOT EXISTS comments (
        id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
        user_id     uuid REFERENCES users(id) ON DELETE CASCADE,
        product_id  uuid REFERENCES products(id) ON DELETE CASCADE,
        comment_text text,
        created_at  timestamptz DEFAULT now()
      );

      CREATE TABLE IF NOT EXISTS likes (
        id         uuid PRIMARY KEY DEFAULT gen_random_uuid(),
        user_id    uuid REFERENCES users(id) ON DELETE CASCADE,
        product_id uuid REFERENCES products(id) ON DELETE CASCADE,
        created_at timestamptz DEFAULT now()
      );
      --------------------------------------------------

   d) Enable Supabase Auth:
      - In the “Authentication” settings, configure email/password signups (or social providers as needed).

3. CONFIGURE ENVIRONMENT VARIABLES  
   - In the newly created Next.js project, create a .env.local file (which is ignored by Git) for storing your Supabase URL and public anon key. For example:
     --------------------------------------------------
     NEXT_PUBLIC_SUPABASE_URL=<your-supabase-url>
     NEXT_PUBLIC_SUPABASE_ANON_KEY=<your-supabase-anon-key>
     --------------------------------------------------

4. CREATE A SUPABASE CLIENT UTILITY (supabase.ts)  
   - In your project, create a file (e.g., lib/supabase.ts) with the following code:
     --------------------------------------------------
     // lib/supabase.ts
     import { createClient } from '@supabase/supabase-js';

     const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!;
     const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!;

     export const supabase = createClient(supabaseUrl, supabaseAnonKey);
     --------------------------------------------------

5. SET UP AUTH PAGES (LOGIN, SIGNUP)  
   - In the pages directory, create pages like /login.tsx and /signup.tsx.  
   - Use Supabase client for signIn and signUp. For example (pseudo-code):
     --------------------------------------------------
     // pages/login.tsx
     import { useState } from 'react';
     import { supabase } from '../lib/supabase';

     export default function LoginPage() {
       const [email, setEmail] = useState('');
       const [password, setPassword] = useState('');

       async function handleLogin() {
         const { error } = await supabase.auth.signInWithPassword({
           email,
           password,
         });
         if (error) console.error(error.message);
         else window.location.href = '/dashboard';
       }

       return (
         <div>
           <h1>Login</h1>
           <input
             type="email"
             placeholder="Email"
             value={email}
             onChange={(e) => setEmail(e.target.value)}
           />
           <input
             type="password"
             placeholder="Password"
             value={password}
             onChange={(e) => setPassword(e.target.value)}
           />
           <button onClick={handleLogin}>Login</button>
         </div>
       );
     }
     --------------------------------------------------

     --------------------------------------------------
     // pages/signup.tsx
     import { useState } from 'react';
     import { supabase } from '../lib/supabase';

     export default function SignupPage() {
       const [email, setEmail] = useState('');
       const [password, setPassword] = useState('');
       const [fullName, setFullName] = useState('');

       async function handleSignup() {
         const { data, error } = await supabase.auth.signUp({
           email,
           password,
         });
         if (error) console.error(error.message);
         // Optionally store fullName in 'users' table if needed
         // using an RPC or after-signup update
       }

       return (
         <div>
           <h1>Sign Up</h1>
           <input
             type="text"
             placeholder="Full Name"
             value={fullName}
             onChange={(e) => setFullName(e.target.value)}
           />
           <input
             type="email"
             placeholder="Email"
             value={email}
             onChange={(e) => setEmail(e.target.value)}
           />
           <input
             type="password"
             placeholder="Password"
             value={password}
             onChange={(e) => setPassword(e.target.value)}
           />
           <button onClick={handleSignup}>Sign Up</button>
         </div>
       );
     }
     --------------------------------------------------

6. CREATE A DASHBOARD FOR OWNERS (PUBLISH PRODUCTS)  
   - Protected route to manage products. For example, /dashboard.tsx:
     --------------------------------------------------
     import { useEffect, useState } from 'react';
     import { supabase } from '../lib/supabase';

     export default function DashboardPage() {
       const [products, setProducts] = useState([]);
       const [title, setTitle] = useState('');
       const [description, setDescription] = useState('');
       const [price, setPrice] = useState('');

       useEffect(() => {
         // Fetch products belonging to the logged-in user
         fetchProducts();
       }, []);

       async function fetchProducts() {
         const { data, error } = await supabase
           .from('products')
           .select('*')
           // Optional: filter by owner_id after obtaining current user's ID
         if (!error && data) setProducts(data);
       }

       async function createProduct() {
         // Get current user
         const {
           data: { user },
         } = await supabase.auth.getUser();
         if (!user) return;

         const { data, error } = await supabase
           .from('products')
           .insert({
             owner_id: user.id,
             title,
             description,
             price: parseFloat(price),
           });
         if (!error) {
           setTitle('');
           setDescription('');
           setPrice('');
           fetchProducts();
         }
       }

       return (
         <div>
           <h1>Owner Dashboard</h1>

           <h2>Create Product</h2>
           <input
             placeholder="Title"
             value={title}
             onChange={(e) => setTitle(e.target.value)}
           />
           <input
             placeholder="Description"
             value={description}
             onChange={(e) => setDescription(e.target.value)}
           />
           <input
             placeholder="Price"
             value={price}
             onChange={(e) => setPrice(e.target.value)}
           />
           <button onClick={createProduct}>Add Product</button>

           <h2>My Products</h2>
           {products.map((product: any) => (
             <div key={product.id}>
               <h3>{product.title}</h3>
               <p>{product.description}</p>
               <p>{product.price}</p>
               {/* TODO: Add edit/delete actions */}
             </div>
           ))}
         </div>
       );
     }
     --------------------------------------------------

7. CREATE A PRODUCT LISTING PAGE & PRODUCT DETAILS  
   - For customers to browse all products. Example: /products/index.tsx, /products/[id].tsx
     --------------------------------------------------
     // pages/products/index.tsx
     import { useEffect, useState } from 'react';
     import { supabase } from '../../lib/supabase';

     export default function ProductsPage() {
       const [products, setProducts] = useState([]);

       useEffect(() => {
         fetchProducts();
       }, []);

       async function fetchProducts() {
         const { data, error } = await supabase
           .from('products')
           .select('*')
           .eq('is_active', true);
         if (!error && data) setProducts(data);
       }

       return (
         <div>
           <h1>All Products</h1>
           {products.map((product: any) => (
             <div key={product.id}>
               <h2>{product.title}</h2>
               <p>{product.description}</p>
               <p>{product.price}</p>
               <a href={`/products/${product.id}`}>View Details</a>
             </div>
           ))}
         </div>
       );
     }
     --------------------------------------------------

     --------------------------------------------------
     // pages/products/[id].tsx
     import { useRouter } from 'next/router';
     import { supabase } from '../../lib/supabase';
     import { useEffect, useState } from 'react';

     export default function ProductDetails() {
       const router = useRouter();
       const { id } = router.query;
       const [product, setProduct] = useState<any>(null);

       useEffect(() => {
         if (id) fetchProduct();
       }, [id]);

       async function fetchProduct() {
         const { data, error } = await supabase
           .from('products')
           .select('*')
           .eq('id', id)
           .single();
         if (!error && data) setProduct(data);
       }

       // Like, comment, or order logic goes here
       // For example:
       // async function handleLike() { ... }

       return (
         <div>
           {product ? (
             <div>
               <h1>{product.title}</h1>
               <p>{product.description}</p>
               <p>Price: {product.price}</p>
               {/* Add comment section, like button, or call to order option */}
             </div>
           ) : (
             <p>Loading...</p>
           )}
         </div>
       );
     }
     --------------------------------------------------

8. OPTIONAL: ADD “LIKE”, “COMMENT”, “ORDER” FUNCTIONALITY  
   - Implement supabase.from('likes'), supabase.from('comments'), and supabase.from('orders') insert operations.  
   - Follow the same structure: create forms or buttons, call your supabase client.  
   - Filter and display user likes or comments in your UI.

9. PROTECT ROUTES & MANAGE USER SESSION  
   - Create a higher-order component or React hook to check if a user is logged in.  
   - If a user goes to /dashboard without being logged in, redirect them to /login.  
   - Optionally, check user role to ensure that only owners can access /dashboard.

10. DEPLOY  
   - Deploy your Next.js app to Vercel or Netlify. Add your environment variables (SUPABASE URL, ANON KEY) in your hosting platform’s settings.  
   - Supabase handles backend hosting automatically.

─────────────────────────────────────────────────────────
(END OF MASTER INSTRUCTION)
─────────────────────────────────────────────────────────

follow the MASTER INSTRUCTION
when you provide the code break it donw if its long it cuts half way before it finish,

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MatiDes12) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
