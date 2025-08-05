حتماً! در ادامه، محتوای مقاله بالا به شکل **کاملاً ساختارمند و مناسب Markdown برای GitHub** نوشته شده است. این نسخه شامل تیترها، کدها و نقل‌قول‌ها با فرمت استاندارد GitHub Markdown است:

---

# Declarative vs Imperative

### Transition into React from JavaScript

**Author: Myung Kim**
*Published: Apr 2, 2019*

---

This article is written for developers transitioning from **vanilla JavaScript** into **React**. Like the author, many are told that *“React is declarative”*, but what does that really mean?

This post breaks down the concept of **declarative vs imperative** programming using metaphors, code samples, and a comparison between vanilla JS and React.

---

## 🧠 Declarative vs Imperative: The Core Idea

From a high-level perspective:

* **Declarative programming** focuses on **WHAT** you want to achieve.
* **Imperative programming** focuses on **HOW** to achieve it.

### 📍 Real-Life Analogy

> **Imperative:**
> “After exiting Port Authority, head southwest on 8th Ave toward W 41st St, turn left onto W 33rd St and you should arrive at Penn Station.”

> **Declarative:**
> “Penn Station is located at 7th & 8th Avenues, between 31st & 33rd Streets New York, NY 10001.”

The imperative version gives step-by-step instructions.
The declarative version describes the destination.

---

## 💻 Code Example: Ruby

### Imperative Version

```ruby
def imperative_search(array, item)
  for i in array do
    if i == item
      return item
    end
  end
  return false
end
```

### Declarative Version

```ruby
def declarative_search(array, item)
  array.find(item)
end
```

The imperative version explains each **step**.
The declarative version simply says **“what to find”**.

---

## ⚛️ React Is Declarative

### Imperative DOM Manipulation (Vanilla JS)

```js
function addArtistNameToBody() {
  const bodyTag = document.querySelector('body')
  const divTag = document.createElement('div')
  let h1Tag = document.createElement('h1')
  h1Tag.innerText = "Mitski"
  divTag.append(h1Tag)
  bodyTag.append(divTag)
}
```

This is very **instructional**, step-by-step, i.e., **imperative**.

---

### Declarative React Version

```jsx
class Artist extends Component {
  render() {
    return (
      <div>
        <h1>{this.props.name}</h1>
      </div>
    )
  }
}
```

Here, we just **describe the desired UI**.
We don’t care how React does it behind the scenes — just **what it should look like**.

> "This is my address. This is how my house looks like. I don’t care how you get here — but get here!"

---

## 🧩 Declarative ≠ Functional Programming

One common confusion:

> Functional programming is a subset of declarative programming, but they are **not** the same thing.

React borrows concepts from functional programming, but its **declarative nature** lies in how it **describes the UI outcome**, not the process.

---

## ✅ Summary

| Imperative       | Declarative                 |
| ---------------- | --------------------------- |
| Focus on **HOW** | Focus on **WHAT**           |
| Gives **steps**  | Gives **result**            |
| Procedural DOM   | JSX UI description          |
| Manual updates   | Virtual DOM handles changes |

---

Hope this helps clarify **why React is considered declarative** — it certainly did for the author.

---

https://medium.com/@myung.kim287/declarative-vs-imperative-251ce99c6c44
