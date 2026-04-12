## loanofficerai-mcp-poc

> description: Apply ALWAYS when implementing authentication features to ensure secure user authentication and session management

description: Apply ALWAYS when implementing authentication features to ensure secure user authentication and session management
globs: "**/*"

# Third-Party Authentication Implementation

This document outlines the requirements and best practices for implementing authentication in applications using established third-party authentication providers.

## Core Requirements

- **REQUIRED**: Use established authentication providers (Auth0, Clerk, NextAuth, etc.) instead of custom solutions
- **REQUIRED**: Follow provider documentation precisely for proper implementation
- **REQUIRED**: Implement proper session validation and refresh mechanisms
- **REQUIRED**: Protect all routes requiring authentication, including API endpoints
- **REQUIRED**: Test authentication flows thoroughly, including edge cases

## Authentication Provider Selection

### Recommended Providers

- **Web Applications**: NextAuth.js, Auth0, Clerk, AWS Cognito, Firebase Authentication
- **Mobile Applications**: Firebase Authentication, Auth0, AWS Amplify
- **Enterprise Applications**: Okta, Azure AD, AWS IAM Identity Center
- **Single-Page Applications**: Auth0, Clerk, Firebase Authentication

### Selection Criteria

- Security track record and compliance certifications
- Support for required authentication methods (passwordless, social, MFA)
- SDK availability for your technology stack
- Pricing and scaling considerations
- User management features
- Documentation quality and community support

## Implementation Guidelines

### Authentication Flow

- Use standardized protocols (OAuth 2.0, OpenID Connect) for authentication
- Implement secure token handling and storage
- Use refresh tokens with appropriate expiration
- Implement proper logout functionality that invalidates tokens
- Consider implementing "remember me" functionality with proper security controls

### Route Protection

- Protect all authenticated routes and API endpoints
- Implement consistent authentication checks across the application
- Use middleware or guards for automatic route protection
- Handle unauthenticated access attempts gracefully
- Redirect users to login when authentication is required

## Implementation Examples

### Next.js with NextAuth.js

```typescript
// GOOD: Setting up NextAuth.js with proper configuration
// pages/api/auth/[...nextauth].ts
import NextAuth from 'next-auth';
import Providers from 'next-auth/providers';

export default NextAuth({
  providers: [
    Providers.Google({
      clientId: process.env.GOOGLE_ID,
      clientSecret: process.env.GOOGLE_SECRET,
    }),
    Providers.Email({
      server: process.env.EMAIL_SERVER,
      from: process.env.EMAIL_FROM,
    }),
  ],
  database: process.env.DATABASE_URL,
  session: {
    jwt: true,
    maxAge: 30 * 24 * 60 * 60, // 30 days
  },
  jwt: {
    secret: process.env.JWT_SECRET,
  },
  pages: {
    signIn: '/auth/signin',
    signOut: '/auth/signout',
    error: '/auth/error',
    verifyRequest: '/auth/verify-request',
  },
  callbacks: {
    async jwt(token, user, account, profile, isNewUser) {
      // Add custom claims to token
      if (user) {
        token.roles = user.roles;
      }
      return token;
    },
    async session(session, token) {
      // Make claims available to client
      session.user.roles = token.roles;
      return session;
    },
  },
});

// GOOD: Protected API route
// pages/api/protected.js
import { getSession } from 'next-auth/client';

export default async (req, res) => {
  const session = await getSession({ req });
  
  if (!session) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  res.status(200).json({ data: 'Protected data' });
};

// GOOD: Protected page component
// pages/protected.js
import { useSession, getSession } from 'next-auth/client';

export default function ProtectedPage() {
  const [session, loading] = useSession();
  
  if (loading) return <div>Loading...</div>;
  
  if (!session) {
    return <div>You must be signed in to view this page</div>;
  }
  
  return <div>Protected content for {session.user.email}</div>;
}

export async function getServerSideProps(context) {
  const session = await getSession(context);
  
  if (!session) {
    return {
      redirect: {
        destination: '/api/auth/signin',
        permanent: false,
      },
    };
  }
  
  return {
    props: { session },
  };
}
```

### React with Auth0

```tsx
// GOOD: Auth0 Provider setup
// src/auth/auth0-provider-with-history.js
import { Auth0Provider } from '@auth0/auth0-react';
import { useNavigate } from 'react-router-dom';

export const Auth0ProviderWithHistory = ({ children }) => {
  const navigate = useNavigate();
  
  const domain = process.env.REACT_APP_AUTH0_DOMAIN;
  const clientId = process.env.REACT_APP_AUTH0_CLIENT_ID;
  const audience = process.env.REACT_APP_AUTH0_AUDIENCE;
  
  const onRedirectCallback = (appState) => {
    navigate(appState?.returnTo || window.location.pathname);
  };
  
  return (
    <Auth0Provider
      domain={domain}
      clientId={clientId}
      redirectUri={window.location.origin}
      onRedirectCallback={onRedirectCallback}
      audience={audience}
    >
      {children}
    </Auth0Provider>
  );
};

// GOOD: Protected route component
// src/components/PrivateRoute.js
import { withAuthenticationRequired } from '@auth0/auth0-react';
import { Route } from 'react-router-dom';
import { LoadingSpinner } from './LoadingSpinner';

export const ProtectedRoute = ({ component, ...args }) => (
  <Route
    component={withAuthenticationRequired(component, {
      onRedirecting: () => <LoadingSpinner />,
    })}
    {...args}
  />
);

// GOOD: Login button
// src/components/LoginButton.js
import { useAuth0 } from '@auth0/auth0-react';

export const LoginButton = () => {
  const { loginWithRedirect } = useAuth0();
  return <button onClick={() => loginWithRedirect()}>Log In</button>;
};
```

## Multi-Factor Authentication (MFA)

### MFA Best Practices

- Offer MFA as an option for all users
- Require MFA for administrative accounts and sensitive operations
- Support multiple second-factor options (authenticator apps, SMS, email)
- Prefer TOTP (Time-based One-Time Password) over SMS
- Implement secure MFA enrollment and recovery processes
- Consider adaptive MFA based on risk factors (location, device, behavior)

### MFA Implementation Example

```typescript
// GOOD: Clerk MFA setup
import { useUser } from '@clerk/nextjs';

function SetupMFAPage() {
  const { user } = useUser();
  
  return (
    <div>
      <h1>Setup MFA</h1>
      {user.twoFactorEnabled ? (
        <div>
          <p>MFA is currently enabled.</p>
          <button onClick={() => user.disableTwoFactor()}>Disable MFA</button>
        </div>
      ) : (
        <div>
          <p>Enhance your account security by enabling MFA.</p>
          <button onClick={() => user.createTwoFactorEnrollment()}>
            Enable MFA
          </button>
        </div>
      )}
    </div>
  );
}
```

## Session Management

### Session Security Requirements

- Use secure, HttpOnly, and SameSite cookies
- Implement proper session timeout and idle timeout
- Support session revocation on logout or security events
- Implement CSRF protection for session cookies
- Refresh tokens securely without exposing them to client-side code
- Consider implementing session binding to client fingerprint

## Security Considerations

- Implement rate limiting on authentication endpoints
- Add CAPTCHA for login attempts and account recovery
- Monitor and log authentication events for security analysis
- Implement secure password reset and account recovery processes
- Handle authentication errors securely without leaking sensitive information
- Update authentication libraries regularly to address security vulnerabilities

## Compliance Requirements

- Maintain documentation of authentication implementation for compliance audits
- Support compliance-specific authentication requirements (e.g., password complexity)
- Implement appropriate privacy controls and consents
- Consider regional requirements for authentication (e.g., SCA for European users)
- Follow industry-specific authentication requirements (e.g., HIPAA, PCI DSS)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spar65)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/spar65)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
