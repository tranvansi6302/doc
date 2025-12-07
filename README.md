# Employee Add & Edit - CustomLocationSelector Integration

## Tổng quan

Đã áp dụng `CustomLocationSelector` cho employee add và edit forms, tương tự như company forms.

## Thay đổi

### 1. Schema Validation (`employee.schema.ts`)

**Thêm mới:**

```typescript
// Thêm cityId field
cityId: z
    .union([z.number(), z.undefined()])
    .refine((val) => val !== undefined && val > 0, {
        message: 'Vui lòng chọn tỉnh/thành phố.'
    }),
```

**Cập nhật:**

```typescript
// countryId - Cho phép undefined
countryId: z
    .union([z.string(), z.undefined()])
    .refine((val) => val !== undefined && /^[A-Z_]+$/.test(val), {
        message: 'Vui lòng chọn quốc gia.'
    }),

// empPlaceOfResidenceWardId - Cho phép undefined
empPlaceOfResidenceWardId: z
    .union([z.number(), z.undefined()])
    .refine((val) => val !== undefined && val > 0, {
        message: 'Vui lòng chọn phường/xã.'
    }),
```

### 2. Employee Add (`employee-add/index.tsx`)

**Import:**

```typescript
import { CustomLocationSelector } from '~/components/customs/cus-location-selector'
```

**Trước:**

```tsx
{/* 2 dropdown riêng lẻ với mock data */}
<CustomDropdownInput
    name='cityId'
    options={[
        { label: 'Hà Nội', value: 1 },
        { label: 'Hồ Chí Minh', value: 2 }
    ]}
/>
<CustomDropdownInput
    name='empPlaceOfResidenceWardId'
    options={[
        { label: 'Phường 1', value: 1 },
        { label: 'Phường 2', value: 2 }
    ]}
/>
```

**Sau:**

```tsx
<CustomLocationSelector
    control={control}
    provinceFieldName='cityId'
    wardFieldName='empPlaceOfResidenceWardId'
    provinceLabel='Tỉnh/Thành phố'
    wardLabel='Phường/Xã'
    provincePlaceholder='Chọn tỉnh/thành phố *'
    wardPlaceholder='Chọn phường/xã *'
    provinceRequired={true}
    wardRequired={true}
    errors={errors}
/>
```

### 3. Employee Edit (`employee-edit/index.tsx`)

**Import:**

```typescript
import { CustomLocationSelector } from '~/components/customs/cus-location-selector'
```

**Reset data - Thêm cityId:**

```typescript
reset({
    // ... other fields
    cityId: d.cityId, // ← THÊM MỚI
    empPlaceOfResidenceWardId: d.empPlaceOfResidenceWardId
    // ... other fields
})
```

**UI - Thay thế dropdown:**

```tsx
{/* Trước: Chỉ có dropdown phường/xã */}
<CustomDropdownInput name='empPlaceOfResidenceWardId' ... />

{/* Sau: Có cả tỉnh/thành phố và phường/xã */}
<CustomLocationSelector
    control={control}
    provinceFieldName='cityId'
    wardFieldName='empPlaceOfResidenceWardId'
    provinceRequired={true}
    wardRequired={true}
    errors={errors}
/>
```

## Field Mapping

| Purpose              | Field Name                   | Type                  | Required |
| -------------------- | ---------------------------- | --------------------- | -------- |
| **Quốc gia**         | `countryId`                  | `string \| undefined` | ✅ Yes   |
| **Tỉnh/Thành phố**   | `cityId`                     | `number \| undefined` | ✅ Yes   |
| **Phường/Xã**        | `empPlaceOfResidenceWardId`  | `number \| undefined` | ✅ Yes   |
| **Địa chỉ chi tiết** | `empPlaceOfResidenceAddress` | `string`              | ✅ Yes   |

## Validation Messages

| Field                        | Error Message                        |
| ---------------------------- | ------------------------------------ |
| `countryId`                  | "Vui lòng chọn quốc gia."            |
| `cityId`                     | "Vui lòng chọn tỉnh/thành phố."      |
| `empPlaceOfResidenceWardId`  | "Vui lòng chọn phường/xã."           |
| `empPlaceOfResidenceAddress` | "Địa chỉ cư trú không được để trống" |

## Flow hoạt động

### Employee Add

```
1. Form load
   ├─ countryId: undefined
   ├─ cityId: undefined
   └─ empPlaceOfResidenceWardId: undefined

2. User chọn quốc gia
   ├─ countryId: "VIETNAM" ✓
   ├─ cityId: undefined
   └─ empPlaceOfResidenceWardId: undefined

3. User chọn tỉnh/thành phố
   ├─ countryId: "VIETNAM" ✓
   ├─ cityId: 1 (Hà Nội) ✓
   ├─ empPlaceOfResidenceWardId: undefined
   └─ Ward dropdown enabled

4. User chọn phường/xã
   ├─ countryId: "VIETNAM" ✓
   ├─ cityId: 1 ✓
   └─ empPlaceOfResidenceWardId: 123 ✓

5. Submit
   └─ All valid ✓
```

### Employee Edit

```
1. API trả về
   {
     countryId: "VIETNAM",
     cityId: 1,
     empPlaceOfResidenceWardId: 123
   }

2. Form load
   ├─ countryId: "VIETNAM" ✓
   ├─ cityId: 1 ✓
   └─ empPlaceOfResidenceWardId: 123 ✓

3. Component render
   ├─ Fetch provinces (wardPid=0)
   ├─ Fetch wards (wardPid=1)
   └─ Hiển thị: "Hà Nội" và "Phường X" ✓

4. User thay đổi tỉnh
   ├─ cityId: 1 → 2
   ├─ empPlaceOfResidenceWardId: 123 → undefined
   └─ Fetch wards mới (wardPid=2) ✓
```

## Benefits

### ✅ Consistency

-   Cùng component với company forms
-   Cùng validation logic
-   Cùng UX pattern

### ✅ Better UX

-   Cascading selection (chọn tỉnh → phường hiện ra)
-   Searchable dropdowns
-   Ward disabled khi chưa chọn tỉnh
-   Clear error messages

### ✅ Real Data

-   Không còn mock data
-   Fetch từ API `/define/WebLocalWardV2`
-   Tự động cập nhật khi có thay đổi

### ✅ Validation

-   Bắt buộc chọn cả 3 fields
-   Error messages tiếng Việt
-   Validation chạy khi submit

## Files Changed

1. ✅ `src/features/hrm/employees/employee.schema.ts`

    - Thêm `cityId` field
    - Cập nhật validation cho `countryId`, `cityId`, `empPlaceOfResidenceWardId`

2. ✅ `src/features/hrm/employees/employee-add/index.tsx`

    - Import `CustomLocationSelector`
    - Thay thế 2 dropdown riêng lẻ bằng `CustomLocationSelector`

3. ✅ `src/features/hrm/employees/employee-edit/index.tsx`
    - Import `CustomLocationSelector`
    - Thêm `cityId` vào reset data
    - Thay thế dropdown phường/xã bằng `CustomLocationSelector`

## Testing

### Test Case 1: Employee Add - Submit trống

```typescript
// Input
{
  countryId: undefined,
  cityId: undefined,
  empPlaceOfResidenceWardId: undefined
}

// Expected
❌ "Vui lòng chọn quốc gia."
❌ "Vui lòng chọn tỉnh/thành phố."
❌ "Vui lòng chọn phường/xã."
```

### Test Case 2: Employee Add - Chọn đầy đủ

```typescript
// Input
{
  countryId: "VIETNAM",
  cityId: 1,
  empPlaceOfResidenceWardId: 123
}

// Expected
✓ All valid
✓ Submit success
```

### Test Case 3: Employee Edit - Load data

```typescript
// API Response
{
  cityId: 1,
  empPlaceOfResidenceWardId: 123
}

// Expected
✓ Form hiển thị: "Hà Nội" và "Phường X"
✓ cityId = 1
✓ empPlaceOfResidenceWardId = 123
```

### Test Case 4: Employee Edit - Thay đổi tỉnh

```typescript
// Initial
cityId: 1, empPlaceOfResidenceWardId: 123

// User chọn tỉnh mới
cityId: 2

// Expected
✓ empPlaceOfResidenceWardId reset về undefined
✓ Fetch wards mới của tỉnh 2
```

## Summary

| Aspect             | Before             | After                   |
| ------------------ | ------------------ | ----------------------- |
| **Province field** | Mock data          | Real API ✅             |
| **Ward field**     | Mock data          | Real API ✅             |
| **Searchable**     | ❌ No              | ✅ Yes                  |
| **Cascading**      | ❌ No              | ✅ Yes                  |
| **Validation**     | Basic              | Advanced ✅             |
| **Error messages** | English            | Vietnamese ✅           |
| **UX**             | Separate dropdowns | Integrated component ✅ |

**Kết quả:** ✅ Employee forms đã có địa chỉ tỉnh/thành phố và phường/xã tương tự company!
