# Ecommerce Permissions — API Mapping

[⬅️ Back to Index](../README.md)

This document maps ecommerce RBAC permissions to the admin APIs that require them and the modules (Brand, Color, Size, etc.) each permission covers.

**Scope:** Only endpoints protected by `@PreAuthorize("hasAuthority('...')")` are listed. Public and customer-facing endpoints (e.g. `/public`, app shopping, place order) use `isAuthenticated()` or no auth and are not included here.

---

## Summary: Modules per Permission

| Permission       | Modules Covered                                              |
|------------------|--------------------------------------------------------------|
| `PRODUCT_VIEW`   | Brand, Color, Size, Category, SubCategory, Product, Stock    |
| `PRODUCT_CREATE` | Brand, Color, Size, Category, SubCategory, Product, Stock    |
| `PRODUCT_EDIT`   | Brand, Color, Size, Category, SubCategory, Product, Stock    |
| `PRODUCT_DELETE` | Brand, Color, Size, Category, SubCategory                  |
| `ORDER_VIEW`     | Order                                                        |
| `ORDER_MANAGE`   | Order                                                        |
| `COUPON_VIEW`    | Coupon                                                       |
| `COUPON_MANAGE`  | Coupon                                                       |

---

## PRODUCT_VIEW

**Modules covered:** Brand, Color, Size, Category, SubCategory, Product, Stock

| Method | Endpoint | Module | Description |
|--------|----------|--------|-------------|
| GET | `/api/v1/sports360/ecommerce/brands/{brandId}` | Brand | Get brand by ID |
| GET | `/api/v1/sports360/ecommerce/brands` | Brand | Get all brands (admin) |
| GET | `/api/v1/sports360/ecommerce/colors/{colorId}` | Color | Get color by ID |
| GET | `/api/v1/sports360/ecommerce/colors` | Color | Get all colors (admin) |
| GET | `/api/v1/sports360/ecommerce/sizes/{sizeId}` | Size | Get size by ID |
| GET | `/api/v1/sports360/ecommerce/sizes` | Size | Get all sizes (admin) |
| GET | `/api/v1/sports360/ecommerce/categories/{categoryId}` | Category | Get category by ID |
| GET | `/api/v1/sports360/ecommerce/categories/paginated` | Category | Get paginated categories |
| GET | `/api/v1/sports360/ecommerce/categories/dropdown` | Category | Get category dropdown |
| GET | `/api/v1/sports360/ecommerce/subcategories/{subCategoryId}` | SubCategory | Get subcategory by ID |
| GET | `/api/v1/sports360/ecommerce/subcategories/paginated` | SubCategory | Get paginated subcategories |
| GET | `/api/v1/sports360/ecommerce/subcategories/dropdown` | SubCategory | Get subcategory dropdown |
| GET | `/api/v1/private/ecommerce/dropdown/add-products` | Product | Dropdown lists for add product |
| GET | `/api/v1/private/ecommerce/products` | Product | Get all products (paginated) |
| GET | `/api/v1/private/ecommerce/products/{productId}/details` | Product | Get product details (admin) |
| GET | `/api/v1/private/ecommerce/products/stock/{productStockId}` | Stock | Get stock details |
| GET | `/api/v1/private/ecommerce/products/stock/thresholds` | Stock | Get low stock alerts |

---

## PRODUCT_CREATE

**Modules covered:** Brand, Color, Size, Category, SubCategory, Product, Stock

| Method | Endpoint | Module | Description |
|--------|----------|--------|-------------|
| POST | `/api/v1/sports360/ecommerce/brands` | Brand | Create brand |
| POST | `/api/v1/sports360/ecommerce/colors` | Color | Create color |
| POST | `/api/v1/sports360/ecommerce/sizes` | Size | Create size |
| POST | `/api/v1/sports360/ecommerce/categories` | Category | Create category |
| POST | `/api/v1/sports360/ecommerce/subcategories` | SubCategory | Create subcategory |
| POST | `/api/v1/private/ecommerce/products` | Product | Create new product (JSON) |
| POST | `/api/v1/private/ecommerce/products/{productId}/stock` | Stock | Add new stock |

---

## PRODUCT_EDIT

**Modules covered:** Brand, Color, Size, Category, SubCategory, Product, Stock

| Method | Endpoint | Module | Description |
|--------|----------|--------|-------------|
| PUT | `/api/v1/sports360/ecommerce/brands/{brandId}` | Brand | Update brand |
| PUT | `/api/v1/sports360/ecommerce/colors/{colorId}` | Color | Update color |
| PUT | `/api/v1/sports360/ecommerce/sizes/{sizeId}` | Size | Update size |
| PUT | `/api/v1/sports360/ecommerce/categories/{categoryId}` | Category | Update category |
| PUT | `/api/v1/sports360/ecommerce/categories/{categoryId}/deactivate` | Category | Deactivate category |
| PUT | `/api/v1/sports360/ecommerce/categories/{categoryId}/activate` | Category | Activate category |
| PUT | `/api/v1/sports360/ecommerce/subcategories/{subCategoryId}` | SubCategory | Update subcategory |
| PUT | `/api/v1/sports360/ecommerce/subcategories/{subCategoryId}/deactivate` | SubCategory | Deactivate subcategory |
| PUT | `/api/v1/sports360/ecommerce/subcategories/{subCategoryId}/activate` | SubCategory | Activate subcategory |
| POST | `/api/v1/private/ecommerce/products/{productId}/discount` | Product | Apply discount |
| PUT | `/api/v1/private/ecommerce/products/{productId}` | Product | Update product (JSON) |
| PUT | `/api/v1/private/ecommerce/products/{productId}/is-active` | Product | Activate/deactivate product |
| PUT | `/api/v1/private/ecommerce/products/{productId}/is-featured` | Product | Update featured tag |
| PUT | `/api/v1/private/ecommerce/products/{productId}/images` | Product | Update product images |
| PUT | `/api/v1/private/ecommerce/products/stock/{productStockId}` | Stock | Update stock (multipart or JSON) |

---

## PRODUCT_DELETE

**Modules covered:** Brand, Color, Size, Category, SubCategory

| Method | Endpoint | Module | Description |
|--------|----------|--------|-------------|
| DELETE | `/api/v1/sports360/ecommerce/brands/{brandId}` | Brand | Delete brand |
| DELETE | `/api/v1/sports360/ecommerce/colors/{colorId}` | Color | Delete color |
| DELETE | `/api/v1/sports360/ecommerce/sizes/{sizeId}` | Size | Delete size |
| DELETE | `/api/v1/sports360/ecommerce/categories/{categoryId}` | Category | Delete category |
| DELETE | `/api/v1/sports360/ecommerce/subcategories/{subCategoryId}` | SubCategory | Delete subcategory |

> **Note:** There is no product delete API gated by `PRODUCT_DELETE` in the codebase today.

---

## ORDER_VIEW

**Modules covered:** Order

| Method | Endpoint | Module | Description |
|--------|----------|--------|-------------|
| GET | `/api/v1/private/ecommerce/orders` | Order | Get all orders (admin, paginated/filtered) |
| GET | `/api/v1/private/ecommerce/orders/{orderId}` | Order | Get order details (admin) |

---

## ORDER_MANAGE

**Modules covered:** Order

| Method | Endpoint | Module | Description |
|--------|----------|--------|-------------|
| PUT | `/api/v1/private/ecommerce/orders/item/{itemId}` | Order | Approve/reject order item |
| PUT | `/api/v1/private/ecommerce/orders/{orderId}/status` | Order | Update order status (query params or JSON) |
| PUT | `/api/v1/private/ecommerce/orders/{orderId}/payment-status` | Order | Update order payment status |

---

## COUPON_VIEW

**Modules covered:** Coupon

| Method | Endpoint | Module | Description |
|--------|----------|--------|-------------|
| GET | `/api/v1/private/ecommerce/coupons` | Coupon | Get paginated coupons |
| GET | `/api/v1/private/ecommerce/coupons/{couponId}` | Coupon | Get coupon details |

---

## COUPON_MANAGE

**Modules covered:** Coupon

| Method | Endpoint | Module | Description |
|--------|----------|--------|-------------|
| POST | `/api/v1/private/ecommerce/coupons` | Coupon | Create coupon |
| PUT | `/api/v1/private/ecommerce/coupons/{couponId}` | Coupon | Update coupon |
| PUT | `/api/v1/private/ecommerce/coupons/{couponId}/is-active` | Coupon | Activate/deactivate coupon |
| DELETE | `/api/v1/private/ecommerce/coupons/{couponId}` | Coupon | Delete coupon |

---

## Notes

1. **Shared product permissions** — `PRODUCT_*` permissions are reused across catalog modules (Brand, Color, Size, Category, SubCategory) as well as Product and Stock. They are not split per module.

## Source Controllers

| Module | Controller |
|--------|------------|
| Brand | `ecommerce/brand/controllers/BrandController.java` |
| Color | `ecommerce/color/controllers/ColorController.java` |
| Size | `ecommerce/size/controllers/SizeController.java` |
| Category | `ecommerce/category/controllers/CategoryController.java` |
| SubCategory | `ecommerce/sub_category/controllers/SubCategoryController.java` |
| Product | `ecommerce/product/controllers/ProductController.java` |
| Stock | `ecommerce/stock/controllers/ProductStockController.java` |
| Order | `ecommerce/order/controllers/CustomerOrderController.java` |
| Coupon | `ecommerce/coupon/controllers/CouponController.java` |
