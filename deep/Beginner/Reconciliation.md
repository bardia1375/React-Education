Certainly! Here is the English translation of your file deep/Beginner/Reconciliation.md:

---

## 1. Example for Reconciliation

Imagine we have a page showing a player's score.

```jsx
function Scoreboard() {
  const [score, setScore] = useState(95);

  return (
    <div>
      <h1>Current score:</h1>
      <p className="score-display">{score}</p>
      <button onClick={() => setScore(score + 1)}>Increase score</button>
    </div>
  );
}
```

When the page loads for the first time, the DOM looks like this:

```html
<div>
  <h1>Current score:</h1>
  <p class="score-display">95</p>
  <button>Increase score</button>
</div>
```

Now, you click the button and `score` becomes `96`.

**What React does **NOT** do:** It does not remove the whole `<div>` and create a new one with `96`.

**What React (with reconciliation) **does**:**
1. Describes a new element tree with the value `96`.
2. Compares it with the previous tree (`95`).
3. Notices `<h1>` and `<button>` are unchanged.
4. Only the content of `<p>` has changed from `95` to `96`.
5. So, it just tells the browser: “Hey browser, find the `<p>` with the class `score-display` and update its text to `96`.”

This means the minimal possible change. This is reconciliation.

---

## 2. Example for Component Identity and State Preservation

Imagine a feedback form where you can switch between "simple" and "advanced" modes.

**Scenario 1: Preserving State**
In both modes, we use an `<input>` component, just changing the `placeholder`.

```jsx
function FeedbackForm() {
  const [isAdvanced, setIsAdvanced] = useState(false);
  
  return (
    <div>
      <label>
        <input type="checkbox" checked={isAdvanced} onChange={() => setIsAdvanced(!isAdvanced)} />
        Advanced mode
      </label>
      <hr />
      {isAdvanced ? (
        <input type="text" placeholder="Please write your detailed feedback..." />
      ) : (
        <input type="text" placeholder="Your feedback..." />
      )}
    </div>
  );
}
```

**What happens:**
1. In "simple" mode, you type “Food was great” in the input.
2. You check "advanced mode".
3. The input still shows your typed text! Only the `placeholder` has changed.
**Why?** Because React sees that in both cases, an `<input>` is at that place, so it preserves the state.

**Scenario 2: Losing State**
Now, in "advanced" mode, we use `<textarea>` instead of `<input>`.

```jsx
// ... (rest of the code is the same)
{isAdvanced ? (
  <textarea placeholder="Please write your detailed feedback..."></textarea>
) : (
  <input type="text" placeholder="Your feedback..." />
)}
```

**What happens:**
1. In "simple" mode, you type “Food was great” in the input.
2. You check "advanced mode".
3. An empty `<textarea>` appears! Your text is gone.
**Why?** Because React sees that the component type switched from `<input>` to `<textarea>`, so it considers them completely different and resets the state.

---

## 3. Example for Element Tree (not Virtual DOM)

Suppose you write this JSX code:

```jsx
const UserCard = ({ name, avatarUrl }) => (
  <div className="card">
    <img src={avatarUrl} className="avatar" />
    <h3>{name}</h3>
  </div>
);
```

When React runs this code, what’s actually created in memory is a simple JS object, not a DOM element:

```javascript
{
  type: 'div',
  props: {
    className: 'card',
    children: [
      {
        type: 'img',
        props: {
          src: 'url-to-avatar.jpg',
          className: 'avatar'
        }
      },
      {
        type: 'h3',
        props: {
          children: 'Bardia'
        }
      }
    ]
  }
}
```

This lightweight, simple structure is the **Element tree**. React can create thousands of these objects in a fraction of a second.

---

## 4. Example for the Magic of `key` (Beyond Lists)

Imagine a chat application where, when you switch user, the draft message for the previous user disappears automatically.

```jsx
function ChatApp({ users }) {
  const [selectedUserId, setSelectedUserId] = useState(users[0].id);

  return (
    <div>
      <UserSelector users={users} onSelect={setSelectedUserId} />
      <hr />
      {/* The magic is here! */}
      <ChatBox key={selectedUserId} userId={selectedUserId} />
    </div>
  );
}

function ChatBox({ userId }) {
  const [draft, setDraft] = useState(''); // Internal state for draft

  // Effect to simulate loading chat history
  useEffect(() => {
    console.log(`Loading chat for user ${userId}...`);
  }, [userId]);

  return (
    <div>
      <p>Chatting with user {userId}</p>
      <textarea 
        value={draft} 
        onChange={(e) => setDraft(e.target.value)}
        placeholder="Type your message..."
      />
    </div>
  );
}
```

**What happens:**
1. You’re chatting with `user-1` and you type “Hi, about tomorrow’s meeting...” in the `<textarea>`.
2. You select `user-2` from the menu.
3. `selectedUserId` changes, and the `key` of the `ChatBox` changes from `user-1` to `user-2`.
4. React sees the new `key` and completely destroys the previous `ChatBox` (with all its state, like the draft).
5. Then, it mounts a **brand new** `ChatBox` with an empty draft.
6. In the console, you see `Loading chat for user user-2...` indicating that `useEffect` runs again and the component is remounted.

With this simple trick, without any extra logic to reset state, your chat form is always clean and ready for the new user.

---

## 5. Example for State Colocation

**Bad Design (State at the top level):**
Suppose we have a dashboard with a user list and a heavy analytics section. The search state is in the main Dashboard component:

```jsx
function Dashboard() {
  const [searchTerm, setSearchTerm] = useState('');
  const visibleUsers = allUsers.filter(u => u.name.includes(searchTerm));

  return (
    <div>
      <h1>Admin Dashboard</h1>
      <SearchInput value={searchTerm} onChange={setSearchTerm} />
      <UserList users={visibleUsers} />
      <HeavyAnalyticsComponent /> {/* This is heavy */}
    </div>
  );
}
```

**Problem:** Every time you type a letter in `SearchInput`, the state changes in `Dashboard`, and **the whole dashboard** rerenders, including the heavy analytics component.

**Good Design (State Colocation):**
Move the search state to a new component that only includes the list and search.

```jsx
function UserManagement() {
  const [searchTerm, setSearchTerm] = useState('');
  const visibleUsers = allUsers.filter(u => u.name.includes(searchTerm));

  return (
    <div>
      <SearchInput value={searchTerm} onChange={setSearchTerm} />
      <UserList users={visibleUsers} />
    </div>
  );
}

function Dashboard() {
  return (
    <div>
      <h1>Admin Dashboard</h1>
      <UserManagement /> {/* Search state is here */}
      <HeavyAnalyticsComponent />
    </div>
  );
}
```

**Result:** Now, when you type in `SearchInput`, only `UserManagement` rerenders. `Dashboard` and, more importantly, `HeavyAnalyticsComponent` remain untouched.

---

## 6. Example for Component Design Optimization

**Bad Design (All-in-one component):**
Suppose we have a product page that manages everything together.

```jsx
function ProductPage({ product }) {
  const [selectedColor, setSelectedColor] = useState('blue');
  const [quantity, setQuantity] = useState(1);
  const [reviews, setReviews] = useState([]);

  useEffect(() => {
    // Suppose we fetch reviews from API
    fetchReviews(product.id).then(setReviews);
  }, [product.id]);

  return (
    <div>
      <ProductDetails product={product} selectedColor={selectedColor} onColorChange={setSelectedColor} />
      <QuantitySelector value={quantity} onChange={setQuantity} />
      <AddToCartButton />
      <ReviewList reviews={reviews} /> {/* Reviews section */}
    </div>
  );
}
```

**Problem:** When the user changes the product color from blue to red, the state in `ProductPage` changes and the whole component rerenders, including the reviews section.

**Good Design (Separation of Concerns):**
Move the reviews section to a separate component with its own logic and state.

```jsx
function ProductConfiguration({ product }) {
  const [selectedColor, setSelectedColor] = useState('blue');
  const [quantity, setQuantity] = useState(1);

  return (
    <div>
      <ProductDetails product={product} selectedColor={selectedColor} onColorChange={setSelectedColor} />
      <QuantitySelector value={quantity} onChange={setQuantity} />
      <AddToCartButton />
    </div>
  );
}

function ReviewsSection({ productId }) {
  const [reviews, setReviews] = useState([]);

  useEffect(() => {
    fetchReviews(productId).then(setReviews);
  }, [productId]);

  return <ReviewList reviews={reviews} />;
}

function ProductPage({ product }) {
  return (
    <div>
      <ProductConfiguration product={product} />
      <hr />
      <ReviewsSection productId={product.id} />
    </div>
  );
}
```

**Result:** Now, when the user changes the product color, only `ProductConfiguration` rerenders. `ReviewsSection` remains completely untouched.

---

If you need the translation in markdown format or want any changes, let me know!
---
---

### ۱. مثال برای Reconciliation (آشتی)

فرض کن یه صفحه داریم که امتیاز یه بازیکن رو نشون میده.

```jsx
function Scoreboard() {
  const [score, setScore] = useState(95);

  return (
    <div>
      <h1>امتیاز فعلی:</h1>
      <p className="score-display">{score}</p>
      <button onClick={() => setScore(score + 1)}>افزایش امتیاز</button>
    </div>
  );
}
```

وقتی صفحه برای اولین بار لود میشه، DOM این شکلیه:

```html
<div>
  <h1>امتیاز فعلی:</h1>
  <p class="score-display">95</p>
  <button>افزایش امتیاز</button>
</div>
```

حالا تو روی دکمه کلیک می‌کنی و `score` میشه `96`.

**کاری که ری‌اکت انجام **نمیده**: ** کل `<div>` رو پاک کنه و از اول یه `<div>` جدید با عدد `96` بسازه.

**کاری که ری‌اکت (با Reconciliation) انجام **میده**: **
1.  یه درخت Element جدید با مقدار `96` توصیف می‌کنه.
2.  اون رو با درخت قبلی (`95`) مقایسه می‌کنه.
3.  متوجه میشه که `<h1>` و `<button>` هیچ تغییری نکردن.
4.  فقط محتوای `<p>` از `95` به `96` تغییر کرده.
5.  پس فقط و فقط یک دستور به مرورگر میده: «ای مرورگر، برو اون `<p>` که کلاس `score-display` داره رو پیدا کن و متنش رو به `96` تغییر بده».

این یعنی حداقل تغییر ممکن. این همون آشتی یا Reconciliation هست.

---

### ۲. مثال برای هویت کامپوننت و حفظ حالت (State)

فرض کن یه فرم نظرخواهی داریم که می‌تونی بین حالت "ساده" و "پیشرفته" سوییچ کنی.

**سناریو ۱: حفظ حالت (State)**
در هر دو حالت از کامپوننت `<input>` استفاده می‌کنیم، فقط `placeholder` رو عوض می‌کنیم.

```jsx
function FeedbackForm() {
  const [isAdvanced, setIsAdvanced] = useState(false);
  
  return (
    <div>
      <label>
        <input type="checkbox" checked={isAdvanced} onChange={() => setIsAdvanced(!isAdvanced)} />
        حالت پیشرفته
      </label>
      <hr />
      {isAdvanced ? (
        <input type="text" placeholder="لطفا نظر دقیق خود را بنویسید..." />
      ) : (
        <input type="text" placeholder="نظر شما..." />
      )}
    </div>
  );
}
```

**اتفاقی که می‌افتد:**
1.  تو حالت "ساده"، توی اینپوت تایپ می‌کنی: «غذا عالی بود».
2.  تیک "حالت پیشرفته" رو می‌زنی.
3.  اینپوت هنوز همون متنی که تایپ کردی رو نشون میده! فقط `placeholder` عوض شده.
**چرا؟** چون ری‌اکت می‌بینه که در هر دو حالت (قبل و بعد از تغییر)، در این جایگاه یک کامپوننت از نوع `<input>` وجود داره. پس میگه «این همونه!» و فقط `props` اون رو (یعنی `placeholder`) آپدیت می‌کنه و به `state` داخلی‌اش (متنی که توش تایپ کردی) دست نمی‌زنه.

**سناریو ۲: از دست رفتن حالت (State)**
حالا در حالت "پیشرفته" به جای `<input>` از `<textarea>` استفاده می‌کنیم.

```jsx
// ... (بقیه کد مثل قبل)
{isAdvanced ? (
  <textarea placeholder="لطفا نظر دقیق خود را بنویسید..."></textarea>
) : (
  <input type="text" placeholder="نظر شما..." />
)}
```

**اتفاقی که می‌افتد:**
1.  تو حالت "ساده"، توی اینپوت تایپ می‌کنی: «غذا عالی بود».
2.  تیک "حالت پیشرفته" رو می‌زنی.
3.  یک `<textarea>` خالی ظاهر میشه! متن تو پاک شد.
**چرا؟** چون ری‌اکت می‌بینه که نوع کامپوننت از `<input>` به `<textarea>` تغییر کرده. پس میگه «اینا دو تا چیز کاملاً متفاوتن!». کامپوننت `<input>` رو با تمام حالتش نابود (unmount) می‌کنه و یه کامپوننت `<textarea>` کاملاً جدید رو از صفر می‌سازه (mount).

---

### ۳. مثال برای درخت Element (نه Virtual DOM)

فرض کن این کد JSX رو می‌نویسی:

```jsx
const UserCard = ({ name, avatarUrl }) => (
  <div className="card">
    <img src={avatarUrl} className="avatar" />
    <h3>{name}</h3>
  </div>
);
```

وقتی ری‌اکت این کد رو اجرا می‌کنه، چیزی که واقعاً در حافظه ساخته میشه، یک آبجکت ساده جاوااسکریپته، نه یک DOM واقعی و سنگین. اون آبجکت این شکلیه:

```javascript
{
  type: 'div',
  props: {
    className: 'card',
    children: [
      {
        type: 'img',
        props: {
          src: 'url-to-avatar.jpg',
          className: 'avatar'
        }
      },
      {
        type: 'h3',
        props: {
          children: 'Bardia'
        }
      }
    ]
  }
}
```

این ساختار سبک و ساده، همون **درخت Element** هست. ری‌اکت می‌تونه هزاران عدد از این آبجکت‌ها رو در کسری از ثانیه با هم مقایسه کنه تا تغییرات رو پیدا کنه، کاری که انجامش روی DOM واقعی مرورگر خیلی کند و پرهزینه است.

---

### ۴. مثال برای جادوی `key` (فراتر از لیست)

فرض کن یه اپلیکیشن چت داری و می‌خوای وقتی کاربر رو عوض می‌کنی، پیش‌نویس پیامی که برای کاربر قبلی می‌نوشتی، پاک بشه و فرم چت کاملاً ریست بشه.

```jsx
function ChatApp({ users }) {
  const [selectedUserId, setSelectedUserId] = useState(users[0].id);

  return (
    <div>
      <UserSelector users={users} onSelect={setSelectedUserId} />
      <hr />
      {/* جادو اینجاست! */}
      <ChatBox key={selectedUserId} userId={selectedUserId} />
    </div>
  );
}

function ChatBox({ userId }) {
  const [draft, setDraft] = useState(''); // State داخلی برای پیش‌نویس

  // یک افکت برای شبیه‌سازی لود کردن تاریخچه چت
  useEffect(() => {
    console.log(`در حال لود کردن چت برای کاربر ${userId}...`);
  }, [userId]);

  return (
    <div>
      <p>در حال چت با کاربر {userId}</p>
      <textarea 
        value={draft} 
        onChange={(e) => setDraft(e.target.value)}
        placeholder="پیام خود را بنویسید..."
      />
    </div>
  );
}
```

**اتفاقی که می‌افتد:**
1.  تو داری با کاربر `user-1` چت می‌کنی و توی `<textarea>` می‌نویسی: «سلام، جلسه فردا...».
2.  از منو، کاربر `user-2` رو انتخاب می‌کنی.
3.  `selectedUserId` تغییر می‌کنه و در نتیجه `key` کامپوننت `ChatBox` از `user-1` به `user-2` تغییر می‌کنه.
4.  ری‌اکت با دیدن `key` جدید، کامپوننت `ChatBox` قبلی رو با تمام `state` هاش (یعنی همون متن پیش‌نویس) **کاملاً نابود می‌کنه (unmount)**.
5.  سپس یک `ChatBox` **کاملاً جدید** از صفر می‌سازه (mount) که `draft` اون یک رشته خالیه.
6.  در کنسول هم لاگ `در حال لود کردن چت برای کاربر user-2...` رو می‌بینی که نشون میده `useEffect` دوباره اجرا شده و کامپوننت جدیده.

با این ترفند ساده، بدون هیچ منطق اضافه‌ای برای ریست کردن state، فرم چت ما همیشه تمیز و آماده برای کاربر جدیده.

---

### ۵. مثال برای هم‌جواری State یا State Colocation

**طراحی بد (State در بالاترین سطح):**
فرض کن یه داشبورد داریم با یه لیست کاربران و یه بخش آمار که خیلی سنگینه. State مربوط به جستجوی کاربران در کامپوننت اصلی (`Dashboard`) قرار داره.

```jsx
function Dashboard() {
  const [searchTerm, setSearchTerm] = useState('');
  const visibleUsers = allUsers.filter(u => u.name.includes(searchTerm));

  return (
    <div>
      <h1>داشبورد مدیریت</h1>
      <SearchInput value={searchTerm} onChange={setSearchTerm} />
      <UserList users={visibleUsers} />
      <HeavyAnalyticsComponent /> {/* این کامپوننت سنگینه */}
    </div>
  );
}
```

**مشکل:** هر بار که یک حرف در `SearchInput` تایپ می‌کنی، `state` در `Dashboard` تغییر می‌کنه و **کل داشبورد** دوباره رندر میشه. این یعنی `HeavyAnalyticsComponent` هم بی‌دلیل دوباره و دوباره رندر میشه و اپلیکیشن کند میشه.

**طراحی خوب (State Colocation):**
ما `state` جستجو رو به یه کامپوننت جدید منتقل می‌کنیم که فقط شامل لیست و جستجو باشه.

```jsx
function UserManagement() {
  const [searchTerm, setSearchTerm] = useState('');
  const visibleUsers = allUsers.filter(u => u.name.includes(searchTerm));

  return (
    <div>
      <SearchInput value={searchTerm} onChange={setSearchTerm} />
      <UserList users={visibleUsers} />
    </div>
  );
}

function Dashboard() {
  return (
    <div>
      <h1>داشبورد مدیریت</h1>
      <UserManagement /> {/* State جستجو داخل این کامپوننته */}
      <HeavyAnalyticsComponent />
    </div>
  );
}
```

**نتیجه:** حالا وقتی در `SearchInput` تایپ می‌کنی، فقط کامپوننت `UserManagement` دوباره رندر میشه. `Dashboard` و مهم‌تر از اون `HeavyAnalyticsComponent` اصلاً متوجه این تغییرات نمیشن و رندر مجدد اتفاق نمی‌افته. این یعنی بهینگی و سرعت بالاتر.

---

### ۶. مثال برای طراحی کامپوننت برای بهینگی

**طراحی بد (کامپوننت همه‌کاره):**
فرض کن یک صفحه محصول داریم که همه چیز رو با هم مدیریت می‌کنه.

```jsx
function ProductPage({ product }) {
  const [selectedColor, setSelectedColor] = useState('blue');
  const [quantity, setQuantity] = useState(1);
  const [reviews, setReviews] = useState([]);

  useEffect(() => {
    // فرض کن نظرات رو از API می‌گیریم
    fetchReviews(product.id).then(setReviews);
  }, [product.id]);

  return (
    <div>
      <ProductDetails product={product} selectedColor={selectedColor} onColorChange={setSelectedColor} />
      <QuantitySelector value={quantity} onChange={setQuantity} />
      <AddToCartButton />
      <ReviewList reviews={reviews} /> {/* بخش نظرات */}
    </div>
  );
}
```

**مشکل:** وقتی کاربر رنگ محصول رو از آبی به قرمز تغییر میده، `state` در `ProductPage` عوض میشه و کل کامپوننت دوباره رندر میشه. این یعنی `ReviewList` که شاید شامل صدها نظر باشه، بی‌دلیل دوباره رندر میشه.

**طراحی خوب (تفکیک مسئولیت‌ها):**
ما بخش نظرات رو به یک کامپوننت جداگانه با منطق و `state` خودش منتقل می‌کنیم.

```jsx
function ProductConfiguration({ product }) {
  const [selectedColor, setSelectedColor] = useState('blue');
  const [quantity, setQuantity] = useState(1);

  return (
    <div>
      <ProductDetails product={product} selectedColor={selectedColor} onColorChange={setSelectedColor} />
      <QuantitySelector value={quantity} onChange={setQuantity} />
      <AddToCartButton />
    </div>
  );
}

function ReviewsSection({ productId }) {
  const [reviews, setReviews] = useState([]);

  useEffect(() => {
    fetchReviews(productId).then(setReviews);
  }, [productId]);

  return <ReviewList reviews={reviews} />;
}

function ProductPage({ product }) {
  return (
    <div>
      <ProductConfiguration product={product} />
      <hr />
      <ReviewsSection productId={product.id} />
    </div>
  );
}
```

**نتیجه:** حالا وقتی کاربر رنگ محصول رو عوض می‌کنه، فقط کامپوننت `ProductConfiguration` دوباره رندر میشه. `ReviewsSection` کاملاً ایزوله است و تا زمانی که `productId` عوض نشه، هیچ رندر مجددی نخواهد داشت. این یعنی ما با **معماری بهتر** به بهینگی رسیدیم، نه با ابزارهایی مثل `React.memo`.
