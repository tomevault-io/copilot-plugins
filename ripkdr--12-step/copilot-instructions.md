## 12-step

> │   ├── app/                    # Next.js App Router

# Web App Development Guidelines

## Next.js 14 App Router Architecture

### Project Structure
```
apps/web/
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── (auth)/            # Authentication pages
│   │   │   ├── layout.tsx     # Auth layout
│   │   │   ├── login/         # Login page
│   │   │   └── register/      # Registration page
│   │   ├── (dashboard)/       # Main dashboard
│   │   │   ├── layout.tsx     # Dashboard layout
│   │   │   ├── page.tsx       # Dashboard home
│   │   │   ├── sponsees/      # Sponsee management
│   │   │   ├── shared-content/ # View shared content
│   │   │   └── settings/      # Sponsor settings
│   │   ├── api/               # API routes
│   │   │   └── trpc/          # tRPC API handler
│   │   ├── globals.css        # Global styles
│   │   ├── layout.tsx         # Root layout
│   │   └── page.tsx           # Landing page
│   ├── components/            # React components
│   │   ├── ui/               # Reusable UI components
│   │   ├── forms/            # Form components
│   │   ├── charts/           # Data visualization
│   │   └── layout/           # Layout components
│   ├── lib/                  # Utilities and configuration
│   │   ├── auth.ts           # NextAuth configuration
│   │   ├── supabase.ts       # Supabase client
│   │   ├── trpc/             # tRPC setup
│   │   └── utils.ts          # Utility functions
│   └── styles/               # Styling files
├── public/                    # Static assets
├── next.config.mjs           # Next.js configuration
└── package.json
```

### Core Technologies
- **Next.js 14**: Latest Next.js with App Router
- **TypeScript**: Strict TypeScript configuration
- **tRPC**: End-to-end typesafe APIs
- **NextAuth.js**: Authentication with Supabase adapter
- **Tailwind CSS**: Utility-first CSS framework
- **React Hook Form + Zod**: Form handling and validation
- **TanStack Query**: Server state management
- **Supabase**: Backend as a Service

## Authentication & Authorization

### NextAuth Configuration
```typescript
// src/lib/auth.ts
import NextAuth from 'next-auth';
import { SupabaseAdapter } from '@auth/supabase-adapter';
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: SupabaseAdapter({
    url: process.env.NEXT_PUBLIC_SUPABASE_URL!,
    secret: process.env.SUPABASE_SERVICE_ROLE_KEY!,
  }),
  providers: [
    {
      id: 'supabase',
      name: 'Supabase',
      type: 'oauth',
      authorization: {
        url: `${process.env.NEXT_PUBLIC_SUPABASE_URL}/auth/v1/authorize`,
        params: {
          provider: 'email',
        },
      },
      token: `${process.env.NEXT_PUBLIC_SUPABASE_URL}/auth/v1/token`,
      userinfo: `${process.env.NEXT_PUBLIC_SUPABASE_URL}/auth/v1/user`,
      clientId: process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
      clientSecret: process.env.SUPABASE_SERVICE_ROLE_KEY!,
    },
  ],
  callbacks: {
    async session({ session, user }) {
      if (session?.user) {
        session.user.id = user.id;
      }
      return session;
    },
    async jwt({ token, user }) {
      if (user) {
        token.id = user.id;
      }
      return token;
    },
  },
  pages: {
    signIn: '/auth/login',
    error: '/auth/error',
  },
});
```

### Route Protection
```typescript
// src/middleware.ts
import { auth } from '@/lib/auth';
import { NextResponse } from 'next/server';

export default auth((req) => {
  const { nextUrl } = req;
  const isLoggedIn = !!req.auth;

  // Protect dashboard routes
  if (nextUrl.pathname.startsWith('/dashboard')) {
    if (!isLoggedIn) {
      return NextResponse.redirect(new URL('/auth/login', nextUrl));
    }
  }

  // Redirect logged-in users from auth pages
  if (nextUrl.pathname.startsWith('/auth') && isLoggedIn) {
    return NextResponse.redirect(new URL('/dashboard', nextUrl));
  }

  return NextResponse.next();
});

export const config = {
  matcher: ['/dashboard/:path*', '/auth/:path*'],
};
```

## tRPC Integration

### Client Setup
```typescript
// src/lib/trpc/client.ts
import { createTRPCReact } from '@trpc/react-query';
import { httpBatchLink } from '@trpc/client';
import type { AppRouter } from '@repo/api';

export const trpc = createTRPCReact<AppRouter>();

export const trpcClient = trpc.createClient({
  links: [
    httpBatchLink({
      url: '/api/trpc',
      headers: async () => {
        const session = await auth();
        return {
          authorization: session?.user?.id ? `Bearer ${session.user.id}` : '',
        };
      },
    }),
  ],
});
```

### React Query Provider
```typescript
// src/lib/trpc/react.tsx
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { useState } from 'react';
import { trpc } from './client';

export function TRPCProvider({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 5 * 60 * 1000, // 5 minutes
        retry: (failureCount, error: any) => {
          if (error.status === 401) return false;
          return failureCount < 3;
        },
      },
    },
  }));

  const [trpcClient] = useState(() => trpc.createClient({
    links: [
      httpBatchLink({
        url: '/api/trpc',
        headers: async () => {
          const session = await auth();
          return {
            authorization: session?.user?.id ? `Bearer ${session.user.id}` : '',
          };
        },
      }),
    ],
  }));

  return (
    <trpc.Provider client={trpcClient} queryClient={queryClient}>
      <QueryClientProvider client={queryClient}>
        {children}
        <ReactQueryDevtools initialIsOpen={false} />
      </QueryClientProvider>
    </trpc.Provider>
  );
}
```

## UI Components

### Design System
```typescript
// src/components/ui/Button.tsx
import { cn } from '@/lib/utils';
import { cva, type VariantProps } from 'class-variance-authority';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:opacity-50 disabled:pointer-events-none ring-offset-background',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input hover:bg-accent hover:text-accent-foreground',
        secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'underline-offset-4 hover:underline text-primary',
      },
      size: {
        default: 'h-10 py-2 px-4',
        sm: 'h-9 px-3 rounded-md',
        lg: 'h-11 px-8 rounded-md',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean;
}

export const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : 'button';
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    );
  }
);
Button.displayName = 'Button';
```

### Form Components
```typescript
// src/components/forms/SponsorCodeForm.tsx
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { Button } from '@/components/ui/Button';
import { Input } from '@/components/ui/Input';
import { trpc } from '@/lib/trpc/client';

const sponsorCodeSchema = z.object({
  code: z.string().min(6, 'Code must be at least 6 characters'),
});

type SponsorCodeFormData = z.infer<typeof sponsorCodeSchema>;

export function SponsorCodeForm() {
  const linkSponsee = trpc.sponsor.linkSponsee.useMutation();

  const form = useForm<SponsorCodeFormData>({
    resolver: zodResolver(sponsorCodeSchema),
    defaultValues: {
      code: '',
    },
  });

  const onSubmit = async (data: SponsorCodeFormData) => {
    try {
      await linkSponsee.mutateAsync({ code: data.code });
      // Handle success
    } catch (error) {
      form.setError('code', {
        message: 'Invalid sponsor code',
      });
    }
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <label htmlFor="code" className="block text-sm font-medium text-gray-700">
          Sponsor Code
        </label>
        <Input
          id="code"
          {...form.register('code')}
          placeholder="Enter sponsor code"
          className="mt-1"
        />
        {form.formState.errors.code && (
          <p className="mt-1 text-sm text-red-600">
            {form.formState.errors.code.message}
          </p>
        )}
      </div>
      
      <Button
        type="submit"
        disabled={form.formState.isSubmitting}
        className="w-full"
      >
        {form.formState.isSubmitting ? 'Linking...' : 'Link with Sponsor'}
      </Button>
    </form>
  );
}
```

## Dashboard Components

### Sponsee Management
```typescript
// src/components/dashboard/SponseeList.tsx
'use client';

import { trpc } from '@/lib/trpc/client';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/Card';
import { Badge } from '@/components/ui/Badge';
import { Button } from '@/components/ui/Button';

export function SponseeList() {
  const { data: sponsees, isLoading } = trpc.sponsor.getSponsees.useQuery();

  if (isLoading) {
    return <div>Loading sponsees...</div>;
  }

  return (
    <div className="space-y-4">
      <h2 className="text-2xl font-bold">Your Sponsees</h2>
      
      {sponsees?.length === 0 ? (
        <Card>
          <CardContent className="p-6 text-center">
            <p className="text-gray-500">No sponsees yet.</p>
            <p className="text-sm text-gray-400 mt-2">
              Share your sponsor code with someone to get started.
            </p>
          </CardContent>
        </Card>
      ) : (
        <div className="grid gap-4">
          {sponsees?.map((sponsee) => (
            <Card key={sponsee.id}>
              <CardHeader>
                <div className="flex items-center justify-between">
                  <CardTitle>{sponsee.handle}</CardTitle>
                  <Badge variant={sponsee.status === 'active' ? 'default' : 'secondary'}>
                    {sponsee.status}
                  </Badge>
                </div>
              </CardHeader>
              <CardContent>
                <div className="space-y-2">
                  <p className="text-sm text-gray-600">
                    Joined: {new Date(sponsee.created_at).toLocaleDateString()}
                  </p>
                  <div className="flex gap-2">
                    <Button size="sm" variant="outline">
                      View Shared Content
                    </Button>
                    <Button size="sm" variant="outline">
                      Send Message
                    </Button>
                  </div>
                </div>
              </CardContent>
            </Card>
          ))}
        </div>
      )}
    </div>
  );
}
```

### Shared Content Viewer
```typescript
// src/components/dashboard/SharedContentViewer.tsx
'use client';

import { trpc } from '@/lib/trpc/client';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/Card';
import { Badge } from '@/components/ui/Badge';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/Tabs';

export function SharedContentViewer({ sponseeId }: { sponseeId: string }) {
  const { data: sharedContent, isLoading } = trpc.sponsor.getSharedContent.useQuery({
    sponseeId,
  });

  if (isLoading) {
    return <div>Loading shared content...</div>;
  }

  return (
    <div className="space-y-6">
      <h2 className="text-2xl font-bold">Shared Content</h2>
      
      <Tabs defaultValue="daily" className="w-full">
        <TabsList>
          <TabsTrigger value="daily">Daily Entries</TabsTrigger>
          <TabsTrigger value="steps">Step Work</TabsTrigger>
          <TabsTrigger value="plans">Action Plans</TabsTrigger>
        </TabsList>
        
        <TabsContent value="daily" className="space-y-4">
          {sharedContent?.dailyEntries?.map((entry) => (
            <Card key={entry.id}>
              <CardHeader>
                <div className="flex items-center justify-between">
                  <CardTitle>{entry.entry_date}</CardTitle>
                  <Badge variant="outline">
                    Cravings: {entry.cravings_intensity}/10
                  </Badge>
                </div>
              </CardHeader>
              <CardContent>
                <div className="space-y-3">
                  {entry.feelings && entry.feelings.length > 0 && (
                    <div>
                      <h4 className="font-medium">Feelings:</h4>
                      <div className="flex flex-wrap gap-1 mt-1">
                        {entry.feelings.map((feeling, index) => (
                          <Badge key={index} variant="secondary">
                            {feeling}
                          </Badge>
                        ))}
                      </div>
                    </div>
                  )}
                  
                  {entry.notes && (
                    <div>
                      <h4 className="font-medium">Notes:</h4>
                      <p className="text-sm text-gray-600 mt-1">{entry.notes}</p>
                    </div>
                  )}
                </div>
              </CardContent>
            </Card>
          ))}
        </TabsContent>
        
        <TabsContent value="steps" className="space-y-4">
          {sharedContent?.stepEntries?.map((entry) => (
            <Card key={entry.id}>
              <CardHeader>
                <CardTitle>Step {entry.step_number}: {entry.title}</CardTitle>
              </CardHeader>
              <CardContent>
                <div className="prose max-w-none">
                  <div dangerouslySetInnerHTML={{ __html: entry.content }} />
                </div>
              </CardContent>
            </Card>
          ))}
        </TabsContent>
        
        <TabsContent value="plans" className="space-y-4">
          {sharedContent?.actionPlans?.map((plan) => (
            <Card key={plan.id}>
              <CardHeader>
                <CardTitle>{plan.title}</CardTitle>
              </CardHeader>
              <CardContent>
                <div className="space-y-3">
                  <div>
                    <h4 className="font-medium">Situation:</h4>
                    <p className="text-sm text-gray-600 mt-1">{plan.situation}</p>
                  </div>
                  
                  {plan.if_then && plan.if_then.length > 0 && (
                    <div>
                      <h4 className="font-medium">If-Then Plans:</h4>
                      <ul className="list-disc list-inside space-y-1 mt-1">
                        {plan.if_then.map((item, index) => (
                          <li key={index} className="text-sm text-gray-600">
                            <strong>If:</strong> {item.if} <strong>Then:</strong> {item.then}
                          </li>
                        ))}
                      </ul>
                    </div>
                  )}
                </div>
              </CardContent>
            </Card>
          ))}
        </TabsContent>
      </Tabs>
    </div>
  );
}
```

## Data Visualization

### Charts and Analytics
```typescript
// src/components/charts/RecoveryChart.tsx
'use client';

import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts';
import { trpc } from '@/lib/trpc/client';

interface RecoveryChartProps {
  sponseeId: string;
  startDate: string;
  endDate: string;
}

export function RecoveryChart({ sponseeId, startDate, endDate }: RecoveryChartProps) {
  const { data: chartData, isLoading } = trpc.sponsor.getSponseeStats.useQuery({
    sponseeId,
    startDate,
    endDate,
  });

  if (isLoading) {
    return <div>Loading chart data...</div>;
  }

  return (
    <div className="h-80 w-full">
      <ResponsiveContainer width="100%" height="100%">
        <LineChart data={chartData}>
          <CartesianGrid strokeDasharray="3 3" />
          <XAxis 
            dataKey="date" 
            tickFormatter={(value) => new Date(value).toLocaleDateString()}
          />
          <YAxis domain={[0, 10]} />
          <Tooltip 
            labelFormatter={(value) => new Date(value).toLocaleDateString()}
            formatter={(value, name) => [value, name === 'cravings' ? 'Cravings' : 'Mood']}
          />
          <Line 
            type="monotone" 
            dataKey="cravings" 
            stroke="#ef4444" 
            strokeWidth={2}
            name="cravings"
          />
        </LineChart>
      </ResponsiveContainer>
    </div>
  );
}
```

## Real-time Features

### WebSocket Integration
```typescript
// src/lib/websocket.ts
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);

export const subscribeToSponseeUpdates = (sponseeId: string, callback: (payload: any) => void) => {
  return supabase
    .channel(`sponsee-${sponseeId}`)
    .on(
      'postgres_changes',
      {
        event: '*',
        schema: 'public',
        table: 'daily_entries',
        filter: `user_id=eq.${sponseeId}`,
      },
      callback
    )
    .subscribe();
};

export const subscribeToMessages = (threadId: string, callback: (payload: any) => void) => {
  return supabase
    .channel(`messages-${threadId}`)
    .on(
      'postgres_changes',
      {
        event: 'INSERT',
        schema: 'public',
        table: 'messages',
        filter: `thread_id=eq.${threadId}`,
      },
      callback
    )
    .subscribe();
};
```

### Real-time Notifications
```typescript
// src/components/notifications/NotificationCenter.tsx
'use client';

import { useEffect, useState } from 'react';
import { subscribeToSponseeUpdates } from '@/lib/websocket';
import { Badge } from '@/components/ui/Badge';
import { Button } from '@/components/ui/Button';

export function NotificationCenter() {
  const [notifications, setNotifications] = useState<Notification[]>([]);
  const [unreadCount, setUnreadCount] = useState(0);

  useEffect(() => {
    // Subscribe to real-time updates
    const subscription = subscribeToSponseeUpdates('current-sponsee-id', (payload) => {
      const notification: Notification = {
        id: Date.now().toString(),
        type: 'daily_entry',
        message: 'New daily entry shared',
        timestamp: new Date(),
        read: false,
      };
      
      setNotifications(prev => [notification, ...prev]);
      setUnreadCount(prev => prev + 1);
    });

    return () => {
      subscription.unsubscribe();
    };
  }, []);

  const markAsRead = (notificationId: string) => {
    setNotifications(prev =>
      prev.map(n => n.id === notificationId ? { ...n, read: true } : n)
    );
    setUnreadCount(prev => Math.max(0, prev - 1));
  };

  return (
    <div className="relative">
      <Button variant="ghost" size="icon">
        <Bell className="h-5 w-5" />
        {unreadCount > 0 && (
          <Badge className="absolute -top-1 -right-1 h-5 w-5 rounded-full p-0 flex items-center justify-center text-xs">
            {unreadCount}
          </Badge>
        )}
      </Button>
    </div>
  );
}
```

## Performance Optimization

### Code Splitting
```typescript
// src/app/dashboard/lazy.tsx
import dynamic from 'next/dynamic';

// Lazy load heavy components
export const SponseeChart = dynamic(
  () => import('@/components/charts/RecoveryChart'),
  { 
    loading: () => <div>Loading chart...</div>,
    ssr: false 
  }
);

export const MessageCenter = dynamic(
  () => import('@/components/messages/MessageCenter'),
  { 
    loading: () => <div>Loading messages...</div> 
  }
);
```

### Image Optimization
```typescript
// src/components/ui/OptimizedImage.tsx
import Image from 'next/image';

interface OptimizedImageProps {
  src: string;
  alt: string;
  width: number;
  height: number;
  className?: string;
}

export function OptimizedImage({ src, alt, width, height, className }: OptimizedImageProps) {
  return (
    <Image
      src={src}
      alt={alt}
      width={width}
      height={height}
      className={className}
      placeholder="blur"
      blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/2wBDAAYEBQYFBAYGBQYHBwYIChAKCgkJChQODwwQFxQYGBcUFhYaHSUfGhsjHBYWICwgIyYnKSopGR8tMC0oMCUoKSj/2wBDAQcHBwoIChMKChMoGhYaKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCj/wAARCAABAAEDASIAAhEBAxEB/8QAFQABAQAAAAAAAAAAAAAAAAAAAAv/xAAUEAEAAAAAAAAAAAAAAAAAAAAA/8QAFQEBAQAAAAAAAAAAAAAAAAAAAAX/xAAUEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwCdABmX/9k="
      sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
    />
  );
}
```

## SEO & Meta

### Metadata Configuration
```typescript
// src/app/layout.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    default: 'Recovery Companion - Sponsor Portal',
    template: '%s | Recovery Companion',
  },
  description: 'Support your sponsees on their recovery journey with our secure sponsor portal.',
  keywords: ['recovery', 'sponsor', '12-step', 'sobriety', 'support'],
  authors: [{ name: 'Recovery Companion Team' }],
  creator: 'Recovery Companion',
  openGraph: {
    type: 'website',
    locale: 'en_US',
    url: 'https://recovery-companion.com',
    title: 'Recovery Companion - Sponsor Portal',
    description: 'Support your sponsees on their recovery journey.',
    siteName: 'Recovery Companion',
  },
  twitter: {
    card: 'summary_large_image',
    title: 'Recovery Companion - Sponsor Portal',
    description: 'Support your sponsees on their recovery journey.',
    creator: '@recoverycompanion',
  },
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      'max-video-preview': -1,
      'max-image-preview': 'large',
      'max-snippet': -1,
    },
  },
};
```

## Error Handling

### Error Boundaries
```typescript
// src/components/ErrorBoundary.tsx
'use client';

import { Component, ReactNode } from 'react';
import { Button } from '@/components/ui/Button';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: any) {
    console.error('Error caught by boundary:', error, errorInfo);
    // Send to Sentry
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="min-h-screen flex items-center justify-center">
          <div className="text-center">
            <h1 className="text-2xl font-bold text-gray-900 mb-4">
              Something went wrong
            </h1>
            <p className="text-gray-600 mb-6">
              We're sorry, but something unexpected happened.
            </p>
            <Button
              onClick={() => this.setState({ hasError: false })}
            >
              Try Again
            </Button>
          </div>
        </div>
      );
    }

    return this.props.children;
  }
}
```

## Testing

### Component Testing
```typescript
// src/__tests__/components/SponsorCodeForm.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { SponsorCodeForm } from '@/components/forms/SponsorCodeForm';
import { trpc } from '@/lib/trpc/client';

// Mock tRPC
jest.mock('@/lib/trpc/client', () => ({
  trpc: {
    sponsor: {
      linkSponsee: {
        useMutation: () => ({
          mutateAsync: jest.fn(),
        }),
      },
    },
  },
}));

describe('SponsorCodeForm', () => {
  it('should submit form with valid code', async () => {
    render(<SponsorCodeForm />);
    
    const codeInput = screen.getByPlaceholderText('Enter sponsor code');
    const submitButton = screen.getByRole('button', { name: /link with sponsor/i });
    
    fireEvent.change(codeInput, { target: { value: 'ABC123' } });
    fireEvent.click(submitButton);
    
    await waitFor(() => {
      expect(trpc.sponsor.linkSponsee.useMutation().mutateAsync).toHaveBeenCalledWith({
        code: 'ABC123',
      });
    });
  });
});
```

### API Testing
```typescript
// src/__tests__/api/trpc.test.ts
import { createMocks } from 'node-mocks-http';
import handler from '@/app/api/trpc/[trpc]/route';

describe('/api/trpc', () => {
  it('should handle tRPC requests', async () => {
    const { req, res } = createMocks({
      method: 'POST',
      url: '/api/trpc/sponsor.getSponsees',
      body: {},
    });

    await handler(req, res);

    expect(res._getStatusCode()).toBe(200);
  });
});
```

## Deployment

### Vercel Configuration
```json
// vercel.json
{
  "functions": {
    "src/app/api/trpc/[trpc]/route.ts": {
      "maxDuration": 30
    }
  },
  "env": {
    "NEXT_PUBLIC_SUPABASE_URL": "@supabase-url",
    "NEXT_PUBLIC_SUPABASE_ANON_KEY": "@supabase-anon-key",
    "SUPABASE_SERVICE_ROLE_KEY": "@supabase-service-role-key"
  }
}
```

### Environment Variables
```bash
# .env.local
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_role_key
NEXTAUTH_SECRET=your_nextauth_secret
NEXTAUTH_URL=http://localhost:3000
```

## Best Practices

### Code Organization
- **Single responsibility**: Each component has one clear purpose
- **Custom hooks**: Extract reusable logic into custom hooks
- **Type safety**: Use TypeScript strictly, avoid `any` types
- **Error handling**: Handle errors gracefully with user-friendly messages
- **Performance**: Use React.memo, useMemo, useCallback appropriately

### User Experience
- **Loading states**: Show loading indicators for async operations
- **Error states**: Provide clear error messages and recovery options
- **Accessibility**: Support screen readers and keyboard navigation
- **Responsive**: Work on different screen sizes
- **Real-time updates**: Use WebSockets for live data

### Security
- **Authentication**: Implement proper session management
- **Authorization**: Check permissions on every request
- **Input validation**: Validate all user inputs
- **CSRF protection**: Use NextAuth's built-in CSRF protection
- **Data privacy**: Respect user privacy and data sharing preferences

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RipKDR) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
