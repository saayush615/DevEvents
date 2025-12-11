# Note-1: Client vs Server Components
**Server Components** = Code that runs on the server (your computer/Vercel's servers). The HTML is generated there and sent to the browser.

**Client Components** = Code that runs in the browser (the user's computer). It can respond to clicks, hovers, and user actions.
> use `use client` to make that component a clinet component, a component is by default a server component

## Where Would I Actually Use This?
_Scenario_: Building an E-Commerce Product Page
Let me show you exactly how your components would be split:
```jsx
// ✅ SERVER COMPONENT (page.tsx)
// Fetches product data from database
const ProductPage = async ({ params }) => {
  // This runs on the SERVER (fast, secure)
  const product = await db.getProduct(params.id)
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      
      {/* ✅ Client Component for user interaction */}
      <AddToCartButton productId={product.id} />
      
      {/* ✅ Server Component for reviews (static data) */}
      <Reviews productId={product.id} />
    </div>
  )
}

// ✅ CLIENT COMPONENT (AddToCartButton.tsx)
'use client'

const AddToCartButton = ({ productId }) => {
  const [isAdding, setIsAdding] = useState(false)
  
  const handleClick = async () => {
    setIsAdding(true) // Show loading spinner
    await addToCart(productId)
    setIsAdding(false)
  }
  
  return (
    <button onClick={handleClick}>
      {isAdding ? 'Adding...' : 'Add to Cart'}
    </button>
  )
}

// ✅ SERVER COMPONENT (Reviews.tsx)
const Reviews = async ({ productId }) => {
  const reviews = await db.getReviews(productId)
  return <ul>{reviews.map(r => <li>{r.text}</li>)}</ul>
}
```
## Why this split?
- **ProductPage** (Server): Fetches data from the database. No need to send API keys to the browser.
- **AddToCartButton** (Client): Needs `onClick` and `useState`. Must run in the browser.
- **Reviews** (Server): Just displays data. No interaction needed.

## The Cheat Sheet: When to Use What?

#### ✅ Use **SERVER Components** When:

| Situation                             | Example                                              |
|---------------------------------------|------------------------------------------------------|
| Fetching data from a database         | `const posts = await db.getPosts()`                  |
| Using API keys or secrets             | `fetch('api.com', { headers: { key: SECRET_KEY } })` |
| No user interaction needed            | A list of blog posts, product descriptions           |
| Reducing JavaScript bundle size       | Heavy libraries (date formatters, markdown parsers)  |


#### ✅ Use **CLIENT Components** When:

| Situation                                 | Example                       |
|-------------------------------------------|-------------------------------|
| Using React hooks                          | `useState`, `useEffect`, `useContext` |
| Event handlers                             | `onClick`, `onChange`, `onSubmit`     |
| Browser APIs                               | `localStorage`, `window`, `document`  |
| Third-party libs needing the DOM           | Charts, animations, modals            |

---