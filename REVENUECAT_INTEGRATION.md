# RevenueCat Integration Documentation

## Overview
This Expo mobile application integrates RevenueCat for subscription management and in-app purchases. The app implements a premium subscription model where users can unlock cat photos by subscribing to a "Premium Cats" entitlement.

## Dependencies

### JavaScript/React Native
- **Package**: `react-native-purchases` (version 8.11.9)
- **Location**: `package.json` line 37
- **Purpose**: React Native wrapper for RevenueCat SDK

### iOS Native Dependencies
- **RevenueCat iOS SDK**: Version 5.32.0
- **PurchasesHybridCommon**: Version 14.2.0 (bridge between React Native and native SDK)
- **Location**: Automatically managed through CocoaPods via `react-native-purchases`

## Configuration

### API Keys Setup
**File**: `app/_layout.tsx` (lines 10-14)

```typescript
if (Platform.OS === "ios") {
  Purchases.configure({ apiKey: "appl_EcbMCqjZgXBWAzYBhshcaJHpjOa" });
} else if (Platform.OS === "android") {
  Purchases.configure({ apiKey: "goog_SOhwxVOHyeCjxadVfIrITqTHMrd" });
}
```

**Configuration Details**:
- **iOS API Key**: `appl_EcbMCqjZgXBWAzYBhshcaJHpjOa`
- **Android API Key**: `goog_SOhwxVOHyeCjxadVfIrITqTHMrd`
- **Initialization**: Occurs in the root layout component's `useEffect` hook
- **Platform Detection**: Uses React Native's `Platform.OS` to configure the appropriate key

## Core Implementation

### 1. Subscription Status Check
**File**: `app/index.tsx` (lines 27-34)

```typescript
/**
 * Retrieves customer information from RevenueCat and checks subscription status
 * Updates local state based on "Premium Cats" entitlement
 */
async function getCustomerInfo() {
  const customerInfo = await Purchases.getCustomerInfo();
  if (typeof customerInfo.entitlements.active["Premium Cats"] !== "undefined") {
    setIsSubscribed(true);
  }
}
```

**Key Features**:
- Checks for active "Premium Cats" entitlement
- Updates UI state to show/hide premium content
- Called on component mount via `useEffect`

### 2. Subscription Purchase Flow
**File**: `app/subscription.tsx` (lines 74-85)

```typescript
/**
 * Handles subscription package purchase
 * @param pkg - RevenueCat package containing product information
 */
const handleSubscribe = async (pkg: PurchasesPackage) => {
  try {
    const { customerInfo } = await Purchases.purchasePackage(pkg);
    if (typeof customerInfo.entitlements.active["Premium Cats"] !== "undefined") {
      router.push("/");
    }
  } catch (e) {
    console.log("📢 error", e);
  }
};
```

**Purchase Process**:
1. User selects a subscription plan
2. `Purchases.purchasePackage()` initiates the purchase
3. Upon successful purchase, checks for "Premium Cats" entitlement
4. Redirects user back to main screen if subscription is active

### 3. Offerings Management
**File**: `app/subscription.tsx` (lines 87-96)

```typescript
/**
 * Fetches available subscription offerings from RevenueCat
 * Updates component state with available packages
 */
async function getOfferings() {
  const offerings = await Purchases.getOfferings();
  if (offerings.current !== null && offerings.current.availablePackages.length !== 0) {
    setOfferings(offerings);
  }
  console.log("📢 offerings", JSON.stringify(offerings, null, 2));
}
```

**Offerings Features**:
- Retrieves current offerings configuration from RevenueCat dashboard
- Validates that offerings exist before updating state
- Dynamically renders subscription plans based on available packages

## UI Implementation

### Premium Content Gating
**File**: `app/index.tsx` (lines 55-67)

The app implements a visual paywall using:
- **Blur Effect**: iOS uses `BlurView`, Android uses semi-transparent overlay
- **Lock Icon**: Ionicons lock symbol overlaid on blurred content
- **Conditional Rendering**: Shows different content based on subscription status

### Subscription Plans Display
**File**: `app/subscription.tsx` (lines 141-155)

```typescript
{offerings?.current?.availablePackages.map((pkg) => (
  <SubscriptionPlan
    key={pkg.identifier}
    title={pkg.product.title}
    price={pkg.product.priceString}
    period={pkg.packageType.toLowerCase()}
    features={[...]}
    onPress={() => handleSubscribe(pkg)}
  />
))}
```

**Dynamic Plan Rendering**:
- Maps through available packages from RevenueCat
- Displays product title, price, and billing period
- Handles purchase initiation on plan selection

## Entitlement System

### Premium Cats Entitlement
- **Entitlement ID**: `"Premium Cats"`
- **Purpose**: Controls access to premium cat photos
- **Check Method**: `customerInfo.entitlements.active["Premium Cats"]`
- **Usage**: Determines UI state and content visibility

## Error Handling

### Purchase Errors
- Basic try-catch implementation in `handleSubscribe`
- Logs errors to console for debugging
- No user-facing error messages currently implemented

### Offerings Validation
- Checks for null offerings before state updates
- Validates available packages array length
- Graceful handling when no offerings are configured

## Development Features

### Debug Toggle
**File**: `app/index.tsx` (lines 105-112)

```typescript
<TouchableOpacity
  style={styles.debugButton}
  onPress={() => setIsSubscribed(!isSubscribed)}
>
  <Text style={styles.debugText}>
    DEBUG: Toggle Subscription ({isSubscribed ? "ON" : "OFF"})
  </Text>
</TouchableOpacity>
```

**Debug Features**:
- Manual subscription state toggle for testing
- Visible debug button in development
- Should be removed in production builds

## File Structure

```
app/
├── _layout.tsx          # RevenueCat configuration and initialization
├── index.tsx            # Main screen with subscription status check
└── subscription.tsx     # Subscription plans and purchase flow

ios/
├── Podfile.lock         # Contains RevenueCat iOS dependencies
└── Pods/
    ├── RevenueCat/      # Native iOS SDK (v5.32.0)
    └── PurchasesHybridCommon/  # React Native bridge (v14.2.0)
```

## Security Considerations

### API Key Exposure
⚠️ **Warning**: API keys are currently hardcoded in the client-side code. For production apps, consider:
- Using environment variables
- Implementing server-side validation
- Using RevenueCat's webhook system for critical business logic

### Entitlement Validation
- Client-side entitlement checks are suitable for UI gating
- Server-side validation recommended for critical features
- RevenueCat provides webhook events for server-side integration

## Best Practices Implemented

1. **Platform-specific Configuration**: Separate API keys for iOS and Android
2. **Error Handling**: Basic error catching in purchase flows
3. **State Management**: Proper React state updates for subscription status
4. **Dynamic Content**: Offerings fetched from RevenueCat dashboard
5. **User Experience**: Visual feedback with blur effects and loading states

## Potential Improvements

1. **Error Handling**: Implement user-facing error messages
2. **Loading States**: Add loading indicators during API calls
3. **Retry Logic**: Handle network failures gracefully
4. **Analytics**: Track subscription events and user behavior
5. **Security**: Move API keys to secure environment variables
6. **Testing**: Add unit tests for subscription logic
7. **Offline Support**: Handle offline scenarios appropriately
