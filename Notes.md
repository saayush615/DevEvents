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
# Note 2: <Link> & <Image> component
## <Link> Component
Remember React Router's `<Link>`? Next.js has its own built-in version that works similarly but with **superpowers**.

#### Key Differences from React Router:
| **Feature**        | **React Router** `<Link>` | **Next.js** `<Link>`                      |
|----------------|------------------------|----------------------------------------|
| **Prefetching**    | ❌ No                  | ✅ Yes (loads page data in background) |
| **Server-Side**    | ❌ Client-only         | ✅ Works with SSR                      |
| **Import From**    | `react-router-dom`     | `next/link`                            |

#### Where would I actually use this?
**Scenario**: You're building a _Netflix clone_. The homepage shows 50 movie cards.

**Without Next.js** `<Link>`: When the user clicks "Movie #1", the browser sends a request to the server, waits, then loads the page. _Feels slow_.

**With Next.js** `<Link>`: When the user just _hovers_ over "Movie #1", Next.js secretly loads that movie's data in the background. When they click, the page appears _instantly_.

## <Image> Component
**Regular HTML** `<img>` tags are like using a **flip phone in 2025**—they work, but they're slow and wasteful.

**Next.js's** `<Image>` is like a **smartphone**—it automatically optimizes, resizes, and lazy-loads images for you.

#### What It Does Automatically:
- **Resizes images** to fit the screen (no loading a 4K image on a phone)
- **Lazy loads** (only loads images when you scroll near them)
- **Converts to modern formats** like WebP (smaller file sizes)
- **Prevents layout shift** (no page jumping when images load)

#### Where would I actually use this?
**Scenario**: You're building an _Instagram clone_ with a feed of user posts.
- **Without** `<Image>`: You load 20 high-res photos at once. The page takes 10 seconds to load, users get frustrated and leave.
- **With** `<Image>`: Only the first 3 visible photos load. As the user scrolls, the rest load one by one. The page loads in 1 second.

#### Real Impact:
- Your logo.png might be 500KB as a raw file
- Next.js <Image> automatically converts it to WebP and shrinks it to 20KB
- 25x smaller file = 25x faster load time

---
