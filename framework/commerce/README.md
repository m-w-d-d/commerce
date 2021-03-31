# Table of Contents

- [BigCommerce Storefront Data Hooks](#bigcommerce-storefront-data-hooks)
  - [Installation](#installation)
  - [General Usage](#general-usage)
    - [CommerceProvider](#commerceprovider)
    - [useLogin hook](#uselogin-hook)
    - [useLogout](#uselogout)
    - [useCustomer](#usecustomer)
    - [useSignup](#usesignup)
    - [usePrice](#useprice)
  - [Cart Hooks](#cart-hooks)
    - [useCart](#usecart)
    - [useAddItem](#useadditem)
    - [useUpdateItem](#useupdateitem)
    - [useRemoveItem](#useremoveitem)
  - [Wishlist Hooks](#wishlist-hooks)
  - [Product Hooks and API](#product-hooks-and-api)
    - [useSearch](#usesearch)
    - [getAllProducts](#getallproducts)
    - [getProduct](#getproduct)
  - [More](#more)

# Commerce Framework

> The core commerce framework is under active development, new features and updates will be continuously added over time. Breaking changes are expected while we finish the API.

The commerce framework ships multiple hooks and a Node.js API, both using an underlying headless e-commerce platform, which we call commerce providers.

The core features are:

- Code splitted hooks for data fetching using [SWR](https://swr.vercel.app/), and to handle common user actions
- A Node.js API for initial data population, static generation of content and for creating the API endpoints that connect to the hooks, if required.

## Commerce Hooks

A commerce hook is a [React hook](https://reactjs.org/docs/hooks-intro.html) that's connected to a commerce provider. They focus on user actions and data fetching of data that wasn't statically generated.

Data fetching hooks use [SWR](https://swr.vercel.app/) underneath and you're welcome to use any of its [return values](https://swr.vercel.app/docs/options#return-values) and [options](https://swr.vercel.app/docs/options#options). For example, using the `useCustomer` hook:

```jsx
const { data, isLoading, error } = useCustomer({
  swrOptions: {
    revalidateOnFocus: true,
  },
})
```

### CommerceProvider

This component adds the provider config and handlers to the context of your React tree for it's children. You can optionally pass the `locale` to it:

```jsx
import { CommerceProvider } from '@framework'

const App = ({ locale = 'en-US', children }) => {
  return <CommerceProvider locale={locale}>{children}</CommerceProvider>
}
```

## Authentication Hooks

### useSignup

Returns a _signup_ function that can be used to sign up the current visitor:

```jsx
import useSignup from '@framework/auth/use-signup'

const SignupView = () => {
  const signup = useSignup()

  const handleSignup = async () => {
    await signup({
      email,
      firstName,
      lastName,
      password,
    })
  }

  return <form onSubmit={handleSignup}>{children}</form>
}
```

### useLogin

Returns a _login_ function that can be used to sign in the current visitor into an existing customer:

```jsx
import useLogin from '@framework/auth/use-login'

const LoginView = () => {
  const login = useLogin()
  const handleLogin = async () => {
    await login({
      email,
      password,
    })
  }

  return <form onSubmit={handleLogin}>{children}</form>
}
```

### useLogout

Returns a _logout_ function that signs out the current customer when called.

```jsx
import useLogout from '@framework/auth/use-logout'

const LogoutButton = () => {
  const logout = useLogout()
  return (
    <button type="button" onClick={() => logout()}>
      Logout
    </button>
  )
}
```

## Customer Hooks

### useCustomer

Fetches and returns the data of the signed in customer:

```jsx
import useCustomer from '@framework/customer/use-customer'

const Profile = () => {
  const { data, isLoading, error } = useCustomer()

  if (isLoading) return <p>Loading...</p>
  if (error) return <p>{error.message}</p>
  if (!data) return null

  return <div>Hello, {data.firstName}</div>
}
```

## Product Hooks

### usePrice

Helper hook to format price according to the commerce locale and currency code. It also handles discounts:

```jsx
import useCart from '@framework/cart/use-cart'
import usePrice from '@framework/product/use-price'

// ...
const { data } = useCart()
const { price, discount, basePrice } = usePrice(
  data && {
    amount: data.subtotalPrice,
    currencyCode: data.currency.code,
    // If `baseAmount` is used, a discount will be calculated
    // baseAmount: number,
  }
)
// ...
```

### useSearch

Fetches and returns the products that match a set of filters:

```jsx
import useSearch from '@framework/product/use-search'

const SearchPage = ({ searchString, category, brand, sortStr }) => {
  const { data } = useSearch({
    search: searchString || '',
    categoryId: category?.entityId,
    brandId: brand?.entityId,
    sort: sortStr,
  })

  return (
    <Grid layout="normal">
      {data.products.map((product) => (
        <ProductCard key={product.path} product={product} />
      ))}
    </Grid>
  )
}
```

## Cart Hooks

### useCart

Returns the current cart data for use

```jsx
...
import useCart from '@bigcommerce/storefront-data-hooks/cart/use-cart'

const countItem = (count: number, item: LineItem) => count + item.quantity

const CartNumber = () => {
  const { data } = useCart()
  const itemsCount = data?.lineItems.reduce(countItem, 0) ?? 0

  return itemsCount > 0 ? <span>{itemsCount}</span> : null
}
```

### useAddItem

```jsx
...
import useAddItem from '@bigcommerce/storefront-data-hooks/cart/use-add-item'

const AddToCartButton = ({ productId, variantId }) => {
  const addItem = useAddItem()

  const addToCart = async () => {
    await addItem({
      productId,
      variantId,
    })
  }

  return <button onClick={addToCart}>Add To Cart</button>
}
...
```

### useUpdateItem

```jsx
...
import useUpdateItem from '@bigcommerce/storefront-data-hooks/cart/use-update-item'

const CartItem = ({ item }) => {
  const [quantity, setQuantity] = useState(item.quantity)
  const updateItem = useUpdateItem(item)

  const updateQuantity = async (e) => {
    const val = e.target.value
    await updateItem({ quantity: val })
  }

  return (
    <input
      type="number"
      max={99}
      min={0}
      value={quantity}
      onChange={updateQuantity}
    />
  )
}
...
```

### useRemoveItem

Provided with a cartItemId, will remove an item from the cart:

```jsx
...
import useRemoveItem from '@bigcommerce/storefront-data-hooks/cart/use-remove-item'

const RemoveButton = ({ item }) => {
  const removeItem = useRemoveItem()

  const handleRemove = async () => {
    await removeItem({ id: item.id })
  }

  return <button onClick={handleRemove}>Remove</button>
}
...
```

## Wishlist Hooks

Wishlist hooks are similar to cart hooks. See the below example for how to use `useWishlist`, `useAddItem`, and `useRemoveItem`.

```jsx
import useAddItem from '@bigcommerce/storefront-data-hooks/wishlist/use-add-item'
import useRemoveItem from '@bigcommerce/storefront-data-hooks/wishlist/use-remove-item'
import useWishlist from '@bigcommerce/storefront-data-hooks/wishlist/use-wishlist'

const WishlistButton = ({ productId, variant }) => {
  const addItem = useAddItem()
  const removeItem = useRemoveItem()
  const { data } = useWishlist()
  const { data: customer } = useCustomer()
  const itemInWishlist = data?.items?.find(
    (item) =>
      item.product_id === productId &&
      item.variant_id === variant?.node.entityId
  )

  const handleWishlistChange = async (e) => {
    e.preventDefault()

    if (!customer) {
      return
    }

    if (itemInWishlist) {
      await removeItem({ id: itemInWishlist.id! })
    } else {
      await addItem({
        productId,
        variantId: variant?.node.entityId!,
      })
    }
  }

  return (
    <button onClick={handleWishlistChange}>
      <Heart fill={itemInWishlist ? 'var(--pink)' : 'none'} />
    </button>
  )
}
```

## Product Hooks and API

### getAllProducts

API function to retrieve a product list.

```js
import { getConfig } from '@bigcommerce/storefront-data-hooks/api'
import getAllProducts from '@bigcommerce/storefront-data-hooks/api/operations/get-all-products'

const { products } = await getAllProducts({
  variables: { field: 'featuredProducts', first: 6 },
  config,
  preview,
})
```

### getProduct

API product to retrieve a single product when provided with the product
slug string.

```js
import { getConfig } from '@bigcommerce/storefront-data-hooks/api'
import getProduct from '@bigcommerce/storefront-data-hooks/api/operations/get-product'

const { product } = await getProduct({
  variables: { slug },
  config,
  preview,
})
```

## More

Feel free to read through the source for more usage, and check the commerce vercel demo and commerce repo for usage examples: ([demo.vercel.store](https://demo.vercel.store/)) ([repo](https://github.com/vercel/commerce))