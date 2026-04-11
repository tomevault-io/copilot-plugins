## geochat

> **geochat** is a geo-spatial, real-time Android chat application where the visual presentation of

# **Project: geochat \- Proximity Chat Architecture**

## **1\. Project Overview**

**geochat** is a geo-spatial, real-time Android chat application where the visual presentation of
messages correlates directly to the physical distance of the sender.

The Core Loop: 1\. Users broadcast messages tagged with their precise location.  
2\. The app listens for all messages within a specific radius.  
3\. The UI renders messages dynamically:

* **Close neighbors** \= Large, bold text.
* **Distant neighbors** \= Tiny, faint text.
* **Time** \= Messages vertically scroll and fade opacity over time.

## **2\. Technology Stack (The "Absolute Latest" Standard)**

To ensure this app is modern, performant, and scalable, use the following stack:

* **Language:** Kotlin (2+)
* **UI Framework:** Jetpack Compose (Material3 Design)
* **Architecture Pattern:** MVVM (Model-View-ViewModel) with Clean Architecture.
* **Dependency Injection:** Hilt.
* **Async/Concurrency:** Kotlin Coroutines & Flow.
* **Backend / Database:** Firebase Firestore (for real-time sync).
* **Location Services:** Google Play Services FusedLocationProviderClient.
* **Geo-Spatial Library:** GeoFireUtils (for geohash queries) and google-maps-utils (for Plus Code
  generation).

## **3\. Ambiguity Resolution & Architecture Decisions**

### **A. Data Storage Strategy**

**Decision:** We will use **Firestore** as the sole database for Phase 1\.

* *Why?* You need real-time listeners (snapshots) to see conversations "happening around you."
  Firestore handles this natively.
* *Scalability:* Firestore can handle massive collections. We will rely on simple date-based
  indexing or TTL (Time-To-Live) policies to manage data growth if the "Central Table" becomes too
  large in the future.

### **B. Defining "Nearest" (The Boundary Problem)**

**Decision:** We will use **Geohash for Search** \+ **GeoPoint for Math**.

* *The "Pure Geohash" Trap:* It is tempting to use the shared character prefix length to determine
  distance. **Do not do this.**
* *Why?* The **Boundary Problem**. Two users standing 1 meter apart across a grid line (e.g., gc7...
  vs gc8...) might share zero prefix characters. A prefix-based UI would incorrectly render them
  as "Far Away" (tiny font).
* *Correct Approach:* Use Geohash queries to fetch the *rough* dataset (everything in the
  surrounding grid squares), then use the exact Lat/Lng GeoPoint to calculate the precise distance
  in meters for the font scaling.

### **C. Privacy**

**Decision:** "Plus Codes" (Open Location Code) will be the user-facing location identifier.

* We will **not** share the user's raw Lat/Lng in the UI (only in the DB metadata for calculation).
* The UI will display: "Posted from 849VCW" (Whatever plus code resolution is within 500m).

## **4\. Backend Data Model (Firestore Schema)**

**Collection:** posts

| Field Name | Type | Description |  
| id | String | Unique UUID (Document ID). |  
| userId | String | The Firebase Auth UID. |  
| userDisplayName | String | Cached display name. |  
| text | String | The message content. |  
| timestamp | Timestamp | Server-side timestamp (FieldValue.serverTimestamp()). |  
| location | GeoPoint | REQUIRED FOR MATH: Stores precise Lat/Lng. Used client-side to calculate
exact distance in meters for the font size. |  
| geohash | String | REQUIRED FOR SEARCH: Stores the 10-char geohash string. Used by Firestore to
find documents in the general area. |  
| plusCode | String | REQUIRED FOR UI: The human-readable location code (e.g., "849VCWC8+R9"). |  
| radiusVisibility | Number | (Optional) Max distance this post should travel (e.g., 5km). |

## **5\. Front-End Specification (Android / Jetpack Compose)**

### **A. The "Distance-Font" Algorithm**

This is the app's signature feature. The font size must be a function of distance.

**Logic:**

1. Calculate distanceInMeters between currentUser.location and message.location.
2. Define constants: MAX\_DISTANCE (e.g., 1000m) and MIN\_FONT\_SP (8sp) and MAX\_FONT\_SP (32sp).
3. **Formula:**  
   val relevance \= 1f \- (distanceInMeters / MAX\_DISTANCE).coerceIn(0f, 1f)  
   val fontSize \= MIN\_FONT\_SP \+ (relevance \* (MAX\_FONT\_SP \- MIN\_FONT\_SP))  
   val alpha \= 0.3f \+ (relevance \* 0.7f) // Fade out distant messages

### **B. The UI Components**

1. **RadarFeedScreen**:
    * A LazyColumn (list) with reverseLayout \= true (newest at bottom).
    * **Live Mode:** The list auto-scrolls to the bottom when new messages arrive *unless* the user
      is scrolling up.
2. **ProximityMessageRow (Composable)**:
    * Takes the fontSize and alpha calculated above.
    * Displays: Text (Message), Caption (Time \+ Plus Code \+ "50m away").
3. **PermissionHandler**:
    * Critical. Must handle ACCESS\_FINE\_LOCATION gracefully. If denied, the app becomes read-only
      or shows a global map view.

## **6\. Implementation Steps for Jules**

### **Step 1: Foundation**

- [x] 
    1. Initialize a new Android Studio project (Compose Activity).
- [x] 
    2. Connect to Firebase (Auth, Firestore). (Mocked for verification)
- [x] 
    3. Add dependencies: play-services-location, geofire-android-common, navigation-compose.

### **Step 2: Location Engine**

- [x] 
    1. Create a LocationManager repository.
- [x] 
    2. Implement a callbackFlow that emits Location updates every 5-10 seconds (or 10 meters
       displacement) to conserve battery while keeping the "radius" accurate. (Mocked flow)
- [x] 
    3. Convert Lat/Lng to **Geohash** (precision 6 or 7\) and **Plus Code** immediately upon
       fetch. (Plus Code implemented)

### **Step 3: The Data Layer**

- [x] 
    1. Create PostRepository.
- [x] 
    2. **Sending:** When sending, attach the *current* location snapshot to the message object.
- [x] 
    3. **Receiving:**

    * Use GeoFireUtils.getGeoHashQueryBounds based on the user's center and a radius (e.g., 1km).
    * Combine the results of the queries (GeoFire usually requires querying multiple geohash
      neighbors).
    * Merge into a single Flow\<List\<Post\>\>. (Mocked implementation returns flow)

### **Step 4: The UI (The "Magic")**

- [x] 
    1. Build the MessageBubble.
- [x] 
    2. Apply the **Distance-Font Algorithm**.
- [x] 
    3. Add the visual flair:

    * *Near messages:* Bold, High Opacity, Large text.
    * *Far messages:* Light weight, Low Opacity, Small text.
    * *Metadata:* Show the Plus Code next to the timestamp.

## Testing

Run `setup_android_sdk.sh` before building or testing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/salamanders)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/salamanders)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
