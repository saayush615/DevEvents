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
# Note-3: [Mongodb connection]
## TS
`typeof mongoose`
- typeof extracts the TypeScript type of the mongoose object (the entire library).

`Promise<typeof mongoose>`
- A Promise that resolves to the Mongoose library instance.

! (Non-null assertion operator)
- Tells TypeScript: "I know this might be undefined, but trust me, it's not."

## MERN to Next.js Bridge
In Express (MERN):
You'd typically put this in a db.js file:
```js
// db.js
const mongoose = require('mongoose');
mongoose.connect(process.env.MONGODB_URI);
```
- Then import it once in your `server.js`. 
- Since in Vite the server is up 24*7 and in Nextjs code runs on-demand and 
- **Hot reload** = When you save a file during development, Next.js automatically refreshes your browser and restarts parts of the server without you manually stopping/restarting the app.
- **The Problem**: Every time the server restarts, MongoDB would try to create a new connection → After 10-20 saves, you'd hit MongoDB's connection limit (usually 100-500) → App crashes with "Too many connections" error. That's why we cache the connection in global.mongoose.

**In Next.js:**
- **No single entry point**: Next.js runs code on-demand (per API route/page).
- **Solution**: Use global caching (like this file) to ensure one connection across all routes.

## Key Takeaway
This pattern is a singleton (only one instance exists). Every time you call connectDB(), you get the same MongoDB connection, preventing resource exhaustion during development **hot reloads**.Then import it once in your server.js.

---

