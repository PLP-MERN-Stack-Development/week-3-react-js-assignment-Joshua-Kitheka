
npm create vite@latest task-manager -- --template react
cd task-manager
npm install

Tailwind CSS Setup:

npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
tailwind.config.js


js
export default {
  content: ["./index.html", "./src/**/*.{js,ts,jsx,tsx}"],
  darkMode: "class",
  theme: {
    extend: {},
  },
  plugins: [],
}
src/index.css


css
@tailwind base;
@tailwind components;
@tailwind utilities;
📂 2. Project Structure
src/
├── assets/
├── components/
│   ├── Button.jsx
│   ├── Card.jsx
│   ├── Navbar.jsx
│   ├── Footer.jsx
│   └── TaskItem.jsx
├── context/
│   └── ThemeContext.jsx
├── hooks/
│   └── useLocalStorage.js
├── layout/
│   └── Layout.jsx
├── pages/
│   ├── Home.jsx
│   └── Tasks.jsx
├── utils/
│   └── api.js
├── App.jsx
└── main.jsx
🧩 3. Components
🔘 Button.jsx
jsx
const Button = ({ children, variant = "primary", ...props }) => {
  const base = "px-4 py-2 rounded font-semibold text-white";
  const styles = {
    primary: "bg-blue-600 hover:bg-blue-700",
    secondary: "bg-gray-500 hover:bg-gray-600",
    danger: "bg-red-600 hover:bg-red-700"
  };
  return (
    <button className={`${base} ${styles[variant]}`} {...props}>
      {children}
    </button>
  );
};
export default Button;
📦 Card.jsx
jsx
const Card = ({ children }) => (
  <div className="p-4 rounded shadow bg-white dark:bg-gray-800">{children}</div>
);
export default Card;
📚 Layout.jsx
jsx
import Navbar from "../components/Navbar";
import Footer from "../components/Footer";

const Layout = ({ children }) => (
  <div className="flex flex-col min-h-screen">
    <Navbar />
    <main className="flex-1 p-4">{children}</main>
    <Footer />
  </div>
);
export default Layout;
🌙 ThemeContext.jsx
jsx
import { createContext, useState, useEffect } from "react";
export const ThemeContext = createContext();

export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState("light");
  useEffect(() => {
    document.documentElement.className = theme;
  }, [theme]);
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme: () => setTheme(prev => prev === "light" ? "dark" : "light") }}>
      {children}
    </ThemeContext.Provider>
  );
};
🧠 4. Task Management & Custom Hook
🪝 useLocalStorage.js
jsx
import { useState, useEffect } from "react";
export function useLocalStorage(key, defaultValue) {
  const [stored, setStored] = useState(() => {
    const saved = localStorage.getItem(key);
    return saved ? JSON.parse(saved) : defaultValue;
  });
  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(stored));
  }, [key, stored]);
  return [stored, setStored];
}
📝 TaskManager.jsx
jsx
import { useState, useEffect } from "react";
import { useLocalStorage } from "../hooks/useLocalStorage";

const TaskManager = () => {
  const [tasks, setTasks] = useLocalStorage("tasks", []);
  const [filter, setFilter] = useState("All");
  const [text, setText] = useState("");

  const filtered = tasks.filter(task =>
    filter === "All" ? true :
    filter === "Active" ? !task.done :
    task.done
  );

  const addTask = () => {
    if (text.trim() === "") return;
    setTasks([...tasks, { id: Date.now(), text, done: false }]);
    setText("");
  };

  const toggleDone = id => {
    setTasks(tasks.map(t => t.id === id ? { ...t, done: !t.done } : t));
  };

  const removeTask = id => {
    setTasks(tasks.filter(t => t.id !== id));
  };

  return (
    <div>
      <div className="flex gap-2 mb-4">
        <input value={text} onChange={e => setText(e.target.value)} className="border px-2 py-1" placeholder="New task" />
        <button onClick={addTask} className="bg-blue-500 text-white px-3">Add</button>
      </div>
      <div className="flex gap-2 mb-2">
        {["All", "Active", "Completed"].map(f => (
          <button key={f} onClick={() => setFilter(f)} className="text-sm underline">
            {f}
          </button>
        ))}
      </div>
      <ul>
        {filtered.map(task => (
          <li key={task.id} className="flex justify-between items-center my-1">
            <span className={task.done ? "line-through" : ""}>{task.text}</span>
            <div className="space-x-2">
              <button onClick={() => toggleDone(task.id)}>✔</button>
              <button onClick={() => removeTask(task.id)}>🗑</button>
            </div>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default TaskManager;
🌐 5. API Integration (Posts Display)
api.js
js
export async function fetchPosts() {
  const res = await fetch("https://jsonplaceholder.typicode.com/posts");
  if (!res.ok) throw new Error("Failed to fetch");
  return await res.json();
}
Posts.jsx
jsx
import { useEffect, useState } from "react";
import { fetchPosts } from "../utils/api";

const Posts = () => {
  const [posts, setPosts] = useState([]);
  const [search, setSearch] = useState("");
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchPosts()
      .then(data => {
        setPosts(data);
        setLoading(false);
      })
      .catch(() => setLoading(false));
  }, []);

  const filtered = posts.filter(p => p.title.includes(search));

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      <input className="border mb-4" value={search} onChange={e => setSearch(e.target.value)} placeholder="Search posts" />
      <div className="grid gap-4 grid-cols-1 sm:grid-cols-2 lg:grid-cols-3">
        {filtered.map(post => (
          <div key={post.id} className="p-4 border rounded">
            <h2 className="font-bold">{post.title}</h2>
            <p>{post.body}</p>
          </div>
        ))}
      </div>
    </div>
  );
};

export default Posts;
📱 6. Responsive & Dark Mode Styling
Add dark classes: bg-white dark:bg-gray-900, text-black dark:text-white

Use Tailwind responsive utilities: sm:, md:, lg:, xl:

Add transitions: transition-all duration-300

Dark Theme Toggle in Navbar:

jsx
const { theme, toggleTheme } = useContext(ThemeContext);
<button onClick={toggleTheme}>
  {theme === "light" ? "🌙" : "☀️"}
</button>
