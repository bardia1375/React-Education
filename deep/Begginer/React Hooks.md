# چرا React Hooks آمد؟ فلسفه و مشکلی که حل کرد

React Hooks در نسخه 16.8 معرفی شد تا سه مشکل اصلی اکوسیستم React را حل کند:

- اشتراک‌گذاری منطق stateful بین کامپوننت‌ها سخت بود
  - با کلاس‌ها مجبور می‌شدیم از HOCها و Render Props استفاده کنیم که به «wrapper hell» منجر می‌شد و کد خوانایی‌اش را از دست می‌داد.
- منطق مرتبط در چرخه‌های حیات پخش می‌شد
  - مثلاً منطق «subscribe» در componentDidMount و «unsubscribe» در componentWillUnmount بود؛ نگهداری و درک آن سخت می‌شد.
- کلاس‌ها پیچیدگی و هزینه ذهنی داشتند
  - this-binding، ترتیب متدها، و بهینه‌سازی‌ها سخت‌تر بود. علاوه بر این، توزیع bundle بهینه و تحلیل استاتیک سخت‌تر می‌شد.

Hooks اجازه می‌دهند بدون کلاس‌ها، در کامپوننت‌های تابعی از state، side-effect، context و سایر قابلیت‌ها استفاده کنیم و منطق قابل‌استفاده‌مجدد را با «custom hook»ها ترکیب‌پذیر کنیم.

---

## مزایا در یک نگاه
- ساده‌سازی کامپوننت‌ها: تابع خالص + Hooks به جای کلاس
- ترکیب و اشتراک‌گذاری منطق: Custom Hooks
- نزدیکی منطق مرتبط: اثر و پاکسازی در یک جا (useEffect)
- بهبود تست‌پذیری و خوانایی
- بهبود تدریجی بدون بازنویسی کل کد

---

## مثال 1: شمارنده قبل و بعد از Hooks

### قبل (کلاس)
```jsx
import React from "react";

class Counter extends React.Component {
  state = { count: 0 };

  componentDidMount() {
    document.title = `Count: ${this.state.count}`;
  }

  componentDidUpdate() {
    document.title = `Count: ${this.state.count}`;
  }

  componentWillUnmount() {
    // پاکسازی در صورت نیاز
  }

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          +
        </button>
      </div>
    );
  }
}

export default Counter;
```

### بعد (تابعی + Hooks)
```jsx
import { useEffect, useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `Count: ${count}`;
    // پاکسازی در صورت نیاز را همین‌جا برگردانید
    return () => {
      // cleanup
    };
  }, [count]);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount((c) => c + 1)}>+</button>
    </div>
  );
}

export default Counter;
```

- منطق مرتبط (به‌روزرسانی title و پاکسازی) کنار هم است.
- نیاز به this و lifecycleهای متعدد نداریم.

---

## مثال 2: اشتراک‌گذاری منطق با Custom Hook

فرض کنید می‌خواهیم state را در localStorage نگه داریم و در چند کامپوننت استفاده کنیم.

```jsx
import { useEffect, useState } from "react";

function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    try {
      const raw = window.localStorage.getItem(key);
      return raw != null ? JSON.parse(raw) : initialValue;
    } catch {
      return initialValue;
    }
  });

  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch {}
  }, [key, value]);

  return [value, setValue];
}

function ThemePicker() {
  const [theme, setTheme] = useLocalStorage("theme", "light");
  return (
    <div>
      <p>Theme: {theme}</p>
      <button onClick={() => setTheme(theme === "light" ? "dark" : "light")}>
        Toggle Theme
      </button>
    </div>
  );
}

function Nickname() {
  const [name, setName] = useLocalStorage("nickname", "guest");
  return (
    <label>
      Nickname: <input value={name} onChange={(e) => setName(e.target.value)} />
    </label>
  );
}
```

- این منطق قبلاً معمولاً با HOC/Render Props پیاده‌سازی می‌شد که تو در تو و سخت‌خوان می‌شد.

---

## مثال 3: نگه‌داشتن منطق مرتبط در یک جا (useEffect + cleanup)

قبلاً: subscribe در DidMount و unsubscribe در WillUnmount. حالا هر دو در یک useEffect کنار هم هستند.

```jsx
import { useEffect, useState } from "react";

function OnlineStatus() {
  const [online, setOnline] = useState(navigator.onLine);

  useEffect(() => {
    const on = () => setOnline(true);
    const off = () => setOnline(false);

    window.addEventListener("online", on);
    window.addEventListener("offline", off);

    // cleanup در همان جا
    return () => {
      window.removeEventListener("online", on);
      window.removeEventListener("offline", off);
    };
  }, []);

  return <p>Status: {online ? "Online" : "Offline"}</p>;
}
```

---

## مثال 4: state پیچیده‌تر با useReducer

برای stateهای چندشاخه/قابل‌پیگیری، useReducer ساختاریافته‌تر از چند useState است.

```jsx
import { useReducer } from "react";

function reducer(state, action) {
  switch (action.type) {
    case "add":
      return { ...state, items: [...state.items, action.item] };
    case "remove":
      return { ...state, items: state.items.filter((i) => i.id !== action.id) };
    case "clear":
      return { ...state, items: [] };
    default:
      return state;
  }
}

function Cart() {
  const [state, dispatch] = useReducer(reducer, { items: [] });

  return (
    <div>
      <button onClick={() => dispatch({ type: "add", item: { id: Date.now() } })}>
        Add
      </button>
      <button onClick={() => dispatch({ type: "clear" })}>Clear</button>
      <ul>
        {state.items.map((i) => (
          <li key={i.id}>
            {i.id}
            <button onClick={() => dispatch({ type: "remove", id: i.id })}>
              x
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## مثال 5: بهینه‌سازی با useMemo و useCallback

```jsx
import { useMemo, useCallback, useState } from "react";

function HeavyList({ items }) {
  const [filter, setFilter] = useState("");

  const filtered = useMemo(() => {
    // محاسبه سنگین
    return items.filter((x) => x.includes(filter));
  }, [items, filter]);

  const onChange = useCallback((e) => setFilter(e.target.value), []);

  return (
    <>
      <input value={filter} onChange={onChange} placeholder="filter" />
      <ul>{filtered.map((x) => <li key={x}>{x}</li>)}</ul>
    </>
  );
}
```

- useMemo از اجرای دوباره محاسبه سنگین جلوگیری می‌کند.
- useCallback از ساخت بیهوده‌ی هندلرهای جدید (و رندر مجدد فرزندانی که به reference حساس‌اند) کم می‌کند.

---

## قواعد طلایی Hooks
- فقط در سطح بالای تابع صدا بزنید (نه داخل شرط/حلقه/توابع داخلی)
- فقط در کامپوننت‌های تابعی یا Custom Hookها استفاده کنید
- وابستگی‌های useEffect/useMemo/useCallback را دقیق تنظیم کنید

---

## جمع‌بندی
Hooks آمد تا:
- اشتراک‌گذاری و ترکیب منطق stateful را ساده کند (Custom Hooks)
- منطق مرتبط را در یک جا نگه دارد (useEffect با cleanup)
- پیچیدگی کلاس‌ها و مشکلات this/lifecycle را کاهش دهد
- مسیر مهاجرت تدریجی و بهترشدن تجربه توسعه را فراهم کند

کلاس‌ها همچنان کار می‌کنند، اما Hooks مسیر توصیه‌شده برای توسعه‌ی React مدرن است.
