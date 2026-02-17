# Numeric Type Conversion Fixes

## Issue
Backend returns numeric values (prices, item values, distances) as strings from MySQL. Using `.toFixed()` directly on these values causes runtime errors.

## Error Examples
```
TypeError: item.estimated_price.toFixed is not a function
TypeError: totalPrice.toFixed is not a function
TypeError: order.price.toFixed is not a function
```

## Root Cause
MySQL DECIMAL fields are returned as strings by the Node.js MySQL driver to preserve precision. JavaScript's `.toFixed()` method only works on Number types.

## Solution
Wrap all numeric values with `Number()` before calling `.toFixed()`:

```typescript
// ❌ Before (causes error)
₦{order.price.toFixed(2)}

// ✅ After (safe)
₦{Number(order.price).toFixed(2)}
```

## Files Fixed

### 1. Cart Store (`web/lib/store/cartStore.ts`)
**Issue**: `getTotalPrice()` could return NaN
```typescript
// Fixed getTotalPrice function
getTotalPrice: () => {
  const { items } = get();
  const total = items.reduce((sum, item) => {
    const price = Number(item.estimated_price) || 0;
    return sum + price;
  }, 0);
  return Number(total) || 0;
}
```

### 2. Cart Summary (`web/components/dashboard/CartSummary.tsx`)
**Issue**: `totalPrice.toFixed()` error
```typescript
// Added safety check
const totalPrice = Number(getTotalPrice()) || 0;
```

### 3. Cart Item (`web/components/dashboard/CartItem.tsx`)
**Issues**: 
- `item.estimated_price.toFixed()` error
- `item.item_value.toLocaleString()` error

**Fixes**:
```typescript
// Estimated price
{item.estimated_price
  ? `₦${Number(item.estimated_price).toFixed(2)}`
  : "Price TBD"}

// Item value badge
{item.item_value && (
  <Badge variant="default">₦{Number(item.item_value).toLocaleString()}</Badge>
)}
```

### 4. Payment Step (`web/components/dashboard/PaymentStep.tsx`)
**Issues**:
- `totalPrice.toFixed()` error  
- `order.price.toFixed()` error

**Fixes**:
```typescript
// Total price
const totalPrice = Number(getTotalPrice()) || 0;

// Order price in success screen
₦{Number(order.price).toFixed(2)}
```

### 5. Order Timeline (`web/components/dashboard/OrderTimeline.tsx`)
**Issue**: `order.price.toFixed()` error
```typescript
// Fixed
₦{Number(order.price).toFixed(2)}
```

### 6. Order Card (`web/components/dashboard/OrderCard.tsx`)
**Issue**: `order.price.toFixed()` error
```typescript
// Fixed
₦{Number(order.price).toFixed(2)}
```

## Pattern to Follow

For all numeric fields from backend, always use:

```typescript
// For prices with decimals
Number(value).toFixed(2)

// For integers with thousand separators
Number(value).toLocaleString()

// For calculations
const total = items.reduce((sum, item) => {
  const price = Number(item.price) || 0;
  return sum + price;
}, 0);
```

## Backend Consideration

If you want to avoid this in the future, you can parse numbers in the backend:

```javascript
// In order.service.js or cart.service.js
const order = {
  ...data,
  price: parseFloat(price),
  item_value: parseFloat(item_value),
  distance_km: parseFloat(distance_km),
  estimated_price: parseFloat(estimated_price)
};
```

However, the frontend Number() wrapper is still recommended as a safety measure.

## Testing Checklist

- ✅ Cart page displays prices correctly
- ✅ Cart summary shows total price
- ✅ Order cards show prices
- ✅ Order timeline displays price
- ✅ Payment step shows totals
- ✅ Item value badges display correctly
- ✅ No runtime errors with .toFixed()
- ✅ Build completes successfully

## Build Status

✅ Frontend build: **Successful**  
✅ TypeScript: **No errors**  
✅ Linting: **No errors**  
✅ All numeric conversions: **Implemented**

---

**Fix Date**: February 17, 2026  
**Status**: Complete - All numeric type issues resolved
