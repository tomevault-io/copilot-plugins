## durp-app

> Technical design for the social scheduling feature.


# 📄 Technical Design Document: Social Scheduling Feature

## 1. Overview

**Goal:** Enhance the NetGains pickleball app with a comprehensive social scheduling feature that allows users to create, join, and manage game sessions with social elements while maintaining the existing pre-play flow.

**Tech Stack:**

- **Frontend:** React Native (Expo)
- **Backend:** Supabase (Postgres, Auth, Edge Functions, Real-time subscriptions)
- **Other:** Google Maps API for location services, Push notifications via Expo

## 2. Requirements

### 2.1 Functional Requirements

#### Core Features:

1. **Session Creation with Social Elements**

   - Extend existing pre-play session creation
   - Add social metadata: visibility (public/friends/private), max players, guest allowance
   - Friend invitation system with multi-select interface
   - Location integration with Google Maps autocomplete
   - DUPR rating filters for session matching

2. **Session Discovery & Joining**

   - Browse public sessions in user's area
   - Filter by date, location, DUPR rating, friend-hosted
   - Join public sessions instantly
   - Real-time updates on session availability

3. **Invitation Management**

   - Send, receive, accept, and reject invitations
   - Track invitation status (pending, accepted, declined)
   - Push notifications for invite events
   - Bulk invite management

4. **Session Participation**
   - View participant lists and status
   - Handle session capacity and waitlists
   - Allow +1 guest system
   - Session cancellation and modification

### 2.2 Non-Functional Requirements

- **Performance:** Sub-2s load times for session lists, real-time updates
- **Platforms:** iOS only (although should be easy to port to Android)
- **Security:** Comprehensive RLS policies, secure friend connections
- **Scalability:** Support for 100+ concurrent users, efficient query patterns
- **Reliability:** 99.9% uptime, graceful error handling

## 3. Architecture

### 3.1 High-Level Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Expo App      │◄──►│   Supabase       │◄──►│  Google Maps    │
│                 │    │                  │    │  API            │
│ • UI Components │    │ • Database       │    │                 │
│ • State Mgmt    │    │ • Auth           │    └─────────────────┘
│ • Navigation    │    │ • Real-time      │
│ • Push Notifs   │    │ • Edge Functions │
└─────────────────┘    │ • Storage        │
                       └──────────────────┘
```

### 3.2 Data Flow

#### Session Creation Flow:

```
User Input → Validation → Session DB Insert → Friend Invitations → Push Notifications → Polling for updates
```

#### Session Discovery Flow:

```
User Location → Session Query (RLS) → Filter/Sort → Display → Join/Request → Update Participants
```

#### Invitation Flow:

```
Invite Creation → Push Notification → User Response → Status Update → Polling for updates
```

## 4. Frontend (Expo)

### 4.1 Project Structure

```
app/
├── (tabs)/
│   └── sessions/
│       ├── _layout.tsx
│       ├── index.tsx           # Main sessions hub
│       ├── create.tsx          # Enhanced session creation
│       ├── browse.tsx          # Public session discovery
│       ├── invites.tsx         # Invitation management
│       └── [sessionId].tsx     # Session detail view
├── components/
│   └── sessions/
│       ├── SessionCard.tsx
│       ├── InviteCard.tsx
│       ├── ParticipantList.tsx
│       ├── LocationPicker.tsx
│       ├── FriendSelector.tsx
│       └── SessionFilters.tsx
└── services/
    ├── sessionService.ts       # Enhanced with social features
    ├── invitationService.ts    # New service
    └── friendService.ts        # New service
```

### 4.2 Core Components

#### Enhanced Session Components:

- **SessionCard:** Display session info with social context
- **InviteCard:** Invitation display with accept/reject actions
- **ParticipantList:** Show session participants and status
- **LocationPicker:** Google Maps integration for location selection
- **FriendSelector:** Multi-select friend invitation interface
- **SessionFilters:** Filter sessions by various criteria

#### New Social Components:

- **SocialSessionForm:** Extended session creation with social fields
- **InvitationManager:** Bulk invitation handling
- **SessionHub:** Central dashboard for all session types

### 4.3 State Management

**Context Providers:**

- **SessionContext:** Global session state and operations
- **InvitationContext:** Invitation state and real-time updates
- **FriendContext:** Friend relationships and selection state

**Real-time Subscriptions:**
(Optional for v1)

- Session participant updates
- Invitation status changes
- New session notifications

### 4.4 Enhanced Auth Flow

```typescript
// Extended auth context with social features
interface AuthContextType {
  user: User | null;
  friends: Friend[];
  invitations: Invitation[];
  signIn: (email: string, password: string) => Promise<void>;
  signOut: () => Promise<void>;
  refreshFriends: () => Promise<void>;
}
```

## 5. Backend (Supabase)

### 5.1 Enhanced Database Schema

#### Current Sessions Table Extensions:

```sql
-- Add social scheduling fields to existing sessions table
ALTER TABLE sessions ADD COLUMN visibility TEXT DEFAULT 'public' CHECK (visibility IN ('public', 'friends', 'private'));
ALTER TABLE sessions ADD COLUMN max_players INTEGER DEFAULT 4;
ALTER TABLE sessions ADD COLUMN allow_guests BOOLEAN DEFAULT false;
ALTER TABLE sessions ADD COLUMN dupr_min DECIMAL(3,1);
ALTER TABLE sessions ADD COLUMN dupr_max DECIMAL(3,1);
ALTER TABLE sessions ADD COLUMN status TEXT DEFAULT 'scheduled' CHECK (status IN ('scheduled', 'cancelled', 'completed'));
```

#### New Tables:

**Friends Table:**

```sql
CREATE TABLE friends (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  friend_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'accepted', 'blocked')),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(user_id, friend_id)
);
```

**Session Participants Table:**

```sql
CREATE TABLE session_participants (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  session_id UUID REFERENCES sessions(id) ON DELETE CASCADE,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  status TEXT DEFAULT 'joined' CHECK (status IN ('invited', 'joined', 'declined', 'waitlisted')),
  is_host BOOLEAN DEFAULT false,
  can_bring_guest BOOLEAN DEFAULT false,
  joined_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(session_id, user_id)
);
```

**Invitations Table:**

```sql
CREATE TABLE invitations (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  session_id UUID REFERENCES sessions(id) ON DELETE CASCADE,
  inviter_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  invitee_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'accepted', 'declined')),
  message TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  responded_at TIMESTAMP WITH TIME ZONE,
  UNIQUE(session_id, invitee_id)
);
```

**User Profiles Table:**

```sql
CREATE TABLE user_profiles (
  id UUID REFERENCES auth.users(id) ON DELETE CASCADE PRIMARY KEY,
  username TEXT UNIQUE,
  display_name TEXT,
  dupr_rating DECIMAL(3,1),

  -- Location using PostGIS with address standardization
  location_coordinates GEOMETRY(POINT, 4326), -- User's home location
  location_raw_address TEXT, -- User-entered address
  location_standardized_address JSONB, -- Standardized address components
  location_radius INTEGER DEFAULT 25, -- miles for session discovery

  push_notifications BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Spatial index for user location queries
CREATE INDEX idx_user_profiles_location_coordinates ON user_profiles USING GIST (location_coordinates);
CREATE INDEX idx_user_profiles_standardized_address ON user_profiles USING GIN (location_standardized_address);
-- Indexes on commonly queried JSONB fields for user profiles
CREATE INDEX idx_user_profiles_location_city ON user_profiles USING GIN ((location_standardized_address->>'city'));
CREATE INDEX idx_user_profiles_location_state ON user_profiles USING GIN ((location_standardized_address->>'state'));
```

### 5.2 Row Level Security (RLS) Policies

#### Sessions Table Policies:

```sql
-- Users can view public sessions or their own sessions
CREATE POLICY "Users can view accessible sessions" ON sessions
  FOR SELECT USING (
    is_public = true OR
    auth.uid() = user_id OR
    auth.uid() IN (
      SELECT user_id FROM session_participants
      WHERE session_id = sessions.id
    )
  );

-- Users can only insert their own sessions
CREATE POLICY "Users can insert their own sessions" ON sessions
  FOR INSERT WITH CHECK (auth.uid() = user_id);

-- Users can update their own sessions or sessions they're hosting
CREATE POLICY "Users can update their sessions" ON sessions
  FOR UPDATE USING (
    auth.uid() = user_id OR
    auth.uid() IN (
      SELECT user_id FROM session_participants
      WHERE session_id = sessions.id AND is_host = true
    )
  );
```

#### Session Participants Policies:

```sql
-- Users can view participants of sessions they can access
CREATE POLICY "Users can view session participants" ON session_participants
  FOR SELECT USING (
    session_id IN (
      SELECT id FROM sessions WHERE
      is_public = true OR
      auth.uid() = user_id OR
      auth.uid() IN (
        SELECT user_id FROM session_participants sp2
        WHERE sp2.session_id = session_participants.session_id
      )
    )
  );

-- Users can join sessions or be added by hosts
CREATE POLICY "Users can join sessions" ON session_participants
  FOR INSERT WITH CHECK (
    auth.uid() = user_id OR
    auth.uid() IN (
      SELECT user_id FROM sessions
      WHERE id = session_id
    )
  );
```

#### Invitations Policies:

```sql
-- Users can view invitations they sent or received
CREATE POLICY "Users can view their invitations" ON invitations
  FOR SELECT USING (
    auth.uid() = inviter_id OR
    auth.uid() = invitee_id
  );

-- Users can send invitations to their own sessions
CREATE POLICY "Users can send invitations" ON invitations
  FOR INSERT WITH CHECK (
    auth.uid() = inviter_id AND
    session_id IN (
      SELECT id FROM sessions WHERE user_id = auth.uid()
    )
  );

-- Users can update invitations they received
CREATE POLICY "Users can respond to invitations" ON invitations
  FOR UPDATE USING (auth.uid() = invitee_id);
```

### 5.3 Edge Functions

#### Session Discovery Function:

```typescript
// supabase/functions/discover-sessions/index.ts
export const corsHeaders = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Headers":
    "authorization, x-client-info, apikey, content-type",
};

serve(async (req) => {
  const { lat, lng, radius, date, dupr_min, dupr_max, city, state } =
    await req.json();

  // Build query with PostGIS spatial functions
  let query = supabase
    .from("sessions")
    .select(
      `
      *,
      session_participants(count),
      user_profiles(display_name, dupr_rating)
    `
    )
    .eq("visibility", "public")
    .eq("date", date);

  // Add DUPR filters if provided
  if (dupr_min) query = query.gte("dupr_min", dupr_min);
  if (dupr_max) query = query.lte("dupr_max", dupr_max);

  // Location filtering - prefer precise coordinates, fallback to city
  if (lat && lng && radius) {
    // Use PostGIS to find sessions within radius
    query = query.rpc("sessions_within_radius", {
      user_lat: lat,
      user_lng: lng,
      radius_meters: radius * 1609.34, // Convert miles to meters
    });
  } else if (city && state) {
    // Fallback to city-based matching using JSONB
    query = query
      .eq("location_standardized_address->city", city)
      .eq("location_standardized_address->state", state);
  }

  const { data, error } = await query;

  return new Response(JSON.stringify({ data, error }), {
    headers: { ...corsHeaders, "Content-Type": "application/json" },
  });
});

// PostGIS function for radius-based session discovery
// This would be created as a Supabase database function:
/*
CREATE OR REPLACE FUNCTION sessions_within_radius(
  user_lat DECIMAL,
  user_lng DECIMAL,
  radius_meters INTEGER
) RETURNS SETOF sessions AS $$
BEGIN
  RETURN QUERY
  SELECT s.*
  FROM sessions s
  WHERE s.location_coordinates IS NOT NULL
    AND ST_DWithin(
      s.location_coordinates::geography,
      ST_Point(user_lng, user_lat)::geography,
      radius_meters
    );
END;
$$ LANGUAGE plpgsql;
*/
```

#### Invitation Notification Function:

```typescript
// supabase/functions/send-invitation/index.ts
serve(async (req) => {
  const { session_id, invitee_ids, message } = await req.json();

  // Create invitations
  const invitations = invitee_ids.map((invitee_id) => ({
    session_id,
    inviter_id: req.headers.get("user-id"),
    invitee_id,
    message,
  }));

  const { error } = await supabase.from("invitations").insert(invitations);

  // Send push notifications
  for (const invitee_id of invitee_ids) {
    await sendPushNotification(invitee_id, {
      title: "New Game Invitation!",
      body: message || "You've been invited to join a pickleball session",
    });
  }

  return new Response(JSON.stringify({ success: !error }));
});
```

## 6. Enhanced Services

### 6.1 Enhanced Session Service

```typescript
// services/sessionService.ts (enhanced)
export const sessionService = {
  // ... existing methods ...

  // Helper function to standardize address
  standardizeAddress: async (rawAddress: string) => {
    if (!rawAddress) return null;

    const { data, error } = await supabase.rpc("standardize_session_address", {
      raw_address: rawAddress,
    });

    if (error) {
      console.error("Address standardization failed:", error);
      return null;
    }

    return data;
  },

  // Create social session with address standardization
  createSocialSession: async (sessionData: SocialSessionData) => {
    // Standardize address if provided
    if (sessionData.location_raw_address) {
      const standardizedAddress = await sessionService.standardizeAddress(
        sessionData.location_raw_address
      );

      if (standardizedAddress) {
        sessionData.location_standardized_address = standardizedAddress;
      }
    }

    const { data: session, error } = await supabase
      .from("sessions")
      .insert({
        ...sessionData,
        user_id: (await supabase.auth.getUser()).data.user?.id,
      })
      .select()
      .single();

    if (error) throw error;

    // Add creator as host participant
    await supabase.from("session_participants").insert({
      session_id: session.id,
      user_id: session.user_id,
      is_host: true,
      status: "joined",
    });

    return session;
  },

  // Update session with address standardization
  updateSession: async (
    sessionId: string,
    updates: Partial<SocialSessionData>
  ) => {
    // Standardize address if being updated
    if (updates.location_raw_address) {
      const standardizedAddress = await sessionService.standardizeAddress(
        updates.location_raw_address
      );

      if (standardizedAddress) {
        updates.location_standardized_address = standardizedAddress;
      }
    }

    const { data, error } = await supabase
      .from("sessions")
      .update(updates)
      .eq("id", sessionId)
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  // Discover public sessions
  discoverSessions: async (filters: SessionFilters) => {
    const { data, error } = await supabase.functions.invoke(
      "discover-sessions",
      {
        body: filters,
      }
    );

    if (error) throw error;
    return data;
  },

  // Join session
  joinSession: async (sessionId: string) => {
    const { data, error } = await supabase
      .from("session_participants")
      .insert({
        session_id: sessionId,
        user_id: (await supabase.auth.getUser()).data.user?.id,
        status: "joined",
      })
      .select()
      .single();

    if (error) throw error;
    return data;
  },
};
```

### 6.2 Invitation Service

```typescript
// services/invitationService.ts (new)
export const invitationService = {
  // Send invitations
  sendInvitations: async (
    sessionId: string,
    inviteeIds: string[],
    message?: string
  ) => {
    const { data, error } = await supabase.functions.invoke("send-invitation", {
      body: { session_id: sessionId, invitee_ids: inviteeIds, message },
    });

    if (error) throw error;
    return data;
  },

  // Get user's invitations
  getInvitations: async (status?: "pending" | "accepted" | "declined") => {
    let query = supabase
      .from("invitations")
      .select(
        `
        *,
        sessions(*),
        inviter:inviter_id(display_name)
      `
      )
      .eq("invitee_id", (await supabase.auth.getUser()).data.user?.id);

    if (status) {
      query = query.eq("status", status);
    }

    const { data, error } = await query.order("created_at", {
      ascending: false,
    });

    if (error) throw error;
    return data;
  },

  // Respond to invitation
  respondToInvitation: async (
    invitationId: string,
    response: "accepted" | "declined"
  ) => {
    const { data, error } = await supabase
      .from("invitations")
      .update({
        status: response,
        responded_at: new Date().toISOString(),
      })
      .eq("id", invitationId)
      .select()
      .single();

    if (error) throw error;

    // If accepted, join the session
    if (response === "accepted") {
      await sessionService.joinSession(data.session_id);
    }

    return data;
  },
};
```

### 6.3 User Profile Service

```typescript
// services/userProfileService.ts (new)
export const userProfileService = {
  // Get user profile
  getProfile: async (userId?: string) => {
    const targetUserId =
      userId || (await supabase.auth.getUser()).data.user?.id;

    const { data, error } = await supabase
      .from("user_profiles")
      .select("*")
      .eq("id", targetUserId)
      .single();

    if (error) throw error;
    return data;
  },

  // Update user profile with address standardization
  updateProfile: async (updates: Partial<UserProfile>) => {
    // Standardize address if being updated
    if (updates.location_raw_address) {
      const standardizedAddress = await sessionService.standardizeAddress(
        updates.location_raw_address
      );

      if (standardizedAddress) {
        updates.location_standardized_address = standardizedAddress;
      }
    }

    const { data, error } = await supabase
      .from("user_profiles")
      .update(updates)
      .eq("id", (await supabase.auth.getUser()).data.user?.id)
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  // Create user profile (for signup)
  createProfile: async (profileData: UserProfileCreate) => {
    // Standardize address if provided
    if (profileData.location_raw_address) {
      const standardizedAddress = await sessionService.standardizeAddress(
        profileData.location_raw_address
      );

      if (standardizedAddress) {
        profileData.location_standardized_address = standardizedAddress;
      }
    }

    const { data, error } = await supabase
      .from("user_profiles")
      .insert({
        ...profileData,
        id: (await supabase.auth.getUser()).data.user?.id,
      })
      .select()
      .single();

    if (error) throw error;
    return data;
  },
};
```

### 6.4 Friend Service

```typescript
// services/friendService.ts (new)
export const friendService = {
  // Get friends
  getFriends: async () => {
    const { data, error } = await supabase
      .from("friends")
      .select(
        `
        *,
        friend:friend_id(id, display_name, dupr_rating)
      `
      )
      .eq("user_id", (await supabase.auth.getUser()).data.user?.id)
      .eq("status", "accepted");

    if (error) throw error;
    return data;
  },

  // Send friend request
  sendFriendRequest: async (friendId: string) => {
    const { data, error } = await supabase
      .from("friends")
      .insert({
        user_id: (await supabase.auth.getUser()).data.user?.id,
        friend_id: friendId,
        status: "pending",
      })
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  // Accept friend request
  acceptFriendRequest: async (friendshipId: string) => {
    const { data, error } = await supabase
      .from("friends")
      .update({ status: "accepted" })
      .eq("id", friendshipId)
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  // Reject friend request
  rejectFriendRequest: async (friendshipId: string) => {
    const { data, error } = await supabase
      .from("friends")
      .update({ status: "rejected" })
      .eq("id", friendshipId)
      .select()
};
```

## 7. Component Architecture

### 7.1 Enhanced Session Components

```typescript
// components/sessions/SocialSessionForm.tsx
interface SocialSessionFormProps {
  onSubmit: (data: SocialSessionData) => void;
  initialData?: Partial<SocialSessionData>;
}

export const SocialSessionForm: React.FC<SocialSessionFormProps> = ({
  onSubmit,
  initialData,
}) => {
  // Form state management
  // Location picker integration
  // Friend selector
  // Public/private toggle
  // Max players input
  // DUPR range selector
};
```

```typescript
// components/sessions/SessionHub.tsx
export const SessionHub: React.FC = () => {
  const [activeTab, setActiveTab] = useState<"upcoming" | "browse" | "invites">(
    "upcoming"
  );

  return (
    <View>
      <TabSelector activeTab={activeTab} onTabChange={setActiveTab} />

      {activeTab === "upcoming" && <UpcomingSessions />}
      {activeTab === "browse" && <BrowseSessions />}
      {activeTab === "invites" && <InvitationManager />}
    </View>
  );
};
```

## 8. Security Considerations

### 8.1 Data Privacy

- Friend connections are mutual and require acceptance
- Private sessions are only visible to invited users
- Location data is only shared with session participants
- Push notification tokens are securely stored

### 8.2 Rate Limiting

- Invitation sending is rate-limited to prevent spam
- Session creation is limited per user per day
- Friend requests are rate-limited

## 9. Performance Optimizations

### 9.1 Database Optimizations

- Spatial indexes for location-based queries
- Composite indexes for common filter combinations
- Query optimization for session discovery
- Connection pooling for high concurrency

### 9.2 Frontend Optimizations

- Lazy loading for session lists
- Image optimization for user avatars
- Caching for frequently accessed data
- Optimistic updates for better UX

### 10. Feature Rollout

1. **Phase 1:** Basic social session creation
2. **Phase 2:** Session discovery and joining
3. **Phase 3:** Advanced invitation management
4. **Phase 4:** Real-time features and notifications

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sabgeetacoe555-collab)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sabgeetacoe555-collab)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
