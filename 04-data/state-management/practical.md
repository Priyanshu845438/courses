
# 🔄 Intermediate State Management

## Table of Contents
- [Advanced Context API](#advanced-context-api)
- [Redux Toolkit](#redux-toolkit)
- [Zustand State Management](#zustand-state-management)
- [State Persistence](#state-persistence)
- [Optimistic Updates](#optimistic-updates)
- [State Normalization](#state-normalization)
- [Async State Management](#async-state-management)
- [State Patterns](#state-patterns)
- [Performance Optimization](#performance-optimization)

---

## ⚛️ Advanced Context API

### Multiple Context Providers

```javascript
// Theme Context
const ThemeContext = createContext();

export const ThemeProvider = ({ children }) => {
    const [theme, setTheme] = useState('light');
    const [primaryColor, setPrimaryColor] = useState('#007bff');
    
    const toggleTheme = () => {
        setTheme(prev => prev === 'light' ? 'dark' : 'light');
    };
    
    const value = {
        theme,
        primaryColor,
        toggleTheme,
        setPrimaryColor
    };
    
    return (
        <ThemeContext.Provider value={value}>
            {children}
        </ThemeContext.Provider>
    );
};

// User Context
const UserContext = createContext();

export const UserProvider = ({ children }) => {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);
    
    const login = async (credentials) => {
        try {
            setLoading(true);
            setError(null);
            const response = await authAPI.login(credentials);
            setUser(response.data.user);
            localStorage.setItem('token', response.data.token);
        } catch (err) {
            setError(err.response?.data?.message || 'Login failed');
        } finally {
            setLoading(false);
        }
    };
    
    const logout = () => {
        setUser(null);
        localStorage.removeItem('token');
    };
    
    const value = {
        user,
        loading,
        error,
        login,
        logout,
        isAuthenticated: !!user
    };
    
    return (
        <UserContext.Provider value={value}>
            {children}
        </UserContext.Provider>
    );
};

// App Context Combination
const AppProviders = ({ children }) => {
    return (
        <ThemeProvider>
            <UserProvider>
                <NotificationProvider>
                    {children}
                </NotificationProvider>
            </UserProvider>
        </ThemeProvider>
    );
};
```

### Context with Reducer

```javascript
// Shopping Cart Context with Reducer
const CartContext = createContext();

const cartReducer = (state, action) => {
    switch (action.type) {
        case 'ADD_ITEM':
            const existingItem = state.items.find(item => item.id === action.payload.id);
            
            if (existingItem) {
                return {
                    ...state,
                    items: state.items.map(item =>
                        item.id === action.payload.id
                            ? { ...item, quantity: item.quantity + 1 }
                            : item
                    )
                };
            }
            
            return {
                ...state,
                items: [...state.items, { ...action.payload, quantity: 1 }]
            };
            
        case 'REMOVE_ITEM':
            return {
                ...state,
                items: state.items.filter(item => item.id !== action.payload.id)
            };
            
        case 'UPDATE_QUANTITY':
            return {
                ...state,
                items: state.items.map(item =>
                    item.id === action.payload.id
                        ? { ...item, quantity: action.payload.quantity }
                        : item
                )
            };
            
        case 'CLEAR_CART':
            return {
                ...state,
                items: []
            };
            
        case 'SET_LOADING':
            return {
                ...state,
                loading: action.payload
            };
            
        default:
            return state;
    }
};

export const CartProvider = ({ children }) => {
    const [state, dispatch] = useReducer(cartReducer, {
        items: [],
        loading: false,
        error: null
    });
    
    const addItem = (item) => {
        dispatch({ type: 'ADD_ITEM', payload: item });
    };
    
    const removeItem = (id) => {
        dispatch({ type: 'REMOVE_ITEM', payload: { id } });
    };
    
    const updateQuantity = (id, quantity) => {
        dispatch({ type: 'UPDATE_QUANTITY', payload: { id, quantity } });
    };
    
    const clearCart = () => {
        dispatch({ type: 'CLEAR_CART' });
    };
    
    const getTotalPrice = () => {
        return state.items.reduce((total, item) => total + (item.price * item.quantity), 0);
    };
    
    const getTotalItems = () => {
        return state.items.reduce((total, item) => total + item.quantity, 0);
    };
    
    const value = {
        ...state,
        addItem,
        removeItem,
        updateQuantity,
        clearCart,
        getTotalPrice,
        getTotalItems
    };
    
    return (
        <CartContext.Provider value={value}>
            {children}
        </CartContext.Provider>
    );
};
```

---

## 🔧 Redux Toolkit

### Store Setup

```javascript
import { configureStore, createSlice } from '@reduxjs/toolkit';
import { persistStore, persistReducer } from 'redux-persist';
import storage from 'redux-persist/lib/storage';

// User Slice
const userSlice = createSlice({
    name: 'user',
    initialState: {
        currentUser: null,
        loading: false,
        error: null
    },
    reducers: {
        loginStart: (state) => {
            state.loading = true;
            state.error = null;
        },
        loginSuccess: (state, action) => {
            state.loading = false;
            state.currentUser = action.payload;
        },
        loginFailure: (state, action) => {
            state.loading = false;
            state.error = action.payload;
        },
        logout: (state) => {
            state.currentUser = null;
            state.error = null;
        },
        updateProfile: (state, action) => {
            state.currentUser = { ...state.currentUser, ...action.payload };
        }
    }
});

// Posts Slice
const postsSlice = createSlice({
    name: 'posts',
    initialState: {
        items: [],
        loading: false,
        error: null,
        currentPost: null
    },
    reducers: {
        fetchPostsStart: (state) => {
            state.loading = true;
            state.error = null;
        },
        fetchPostsSuccess: (state, action) => {
            state.loading = false;
            state.items = action.payload;
        },
        fetchPostsFailure: (state, action) => {
            state.loading = false;
            state.error = action.payload;
        },
        addPost: (state, action) => {
            state.items.unshift(action.payload);
        },
        updatePost: (state, action) => {
            const index = state.items.findIndex(post => post.id === action.payload.id);
            if (index !== -1) {
                state.items[index] = action.payload;
            }
        },
        deletePost: (state, action) => {
            state.items = state.items.filter(post => post.id !== action.payload);
        },
        setCurrentPost: (state, action) => {
            state.currentPost = action.payload;
        }
    }
});

// Persist configuration
const persistConfig = {
    key: 'root',
    storage,
    whitelist: ['user'] // Only persist user data
};

const persistedReducer = persistReducer(persistConfig, userSlice.reducer);

// Store configuration
export const store = configureStore({
    reducer: {
        user: persistedReducer,
        posts: postsSlice.reducer
    },
    middleware: (getDefaultMiddleware) =>
        getDefaultMiddleware({
            serializableCheck: {
                ignoredActions: ['persist/PERSIST', 'persist/REHYDRATE']
            }
        })
});

export const persistor = persistStore(store);
export const { loginStart, loginSuccess, loginFailure, logout, updateProfile } = userSlice.actions;
export const { fetchPostsStart, fetchPostsSuccess, fetchPostsFailure, addPost, updatePost, deletePost, setCurrentPost } = postsSlice.actions;
```

### Async Thunks

```javascript
import { createAsyncThunk } from '@reduxjs/toolkit';

// Async thunk for login
export const loginUser = createAsyncThunk(
    'user/login',
    async (credentials, { rejectWithValue }) => {
        try {
            const response = await authAPI.login(credentials);
            localStorage.setItem('token', response.data.token);
            return response.data.user;
        } catch (error) {
            return rejectWithValue(error.response?.data?.message || 'Login failed');
        }
    }
);

// Async thunk for fetching posts
export const fetchPosts = createAsyncThunk(
    'posts/fetchPosts',
    async (params, { rejectWithValue }) => {
        try {
            const response = await postsAPI.getPosts(params);
            return response.data;
        } catch (error) {
            return rejectWithValue(error.response?.data?.message || 'Failed to fetch posts');
        }
    }
);

// Async thunk for creating post
export const createPost = createAsyncThunk(
    'posts/createPost',
    async (postData, { rejectWithValue }) => {
        try {
            const response = await postsAPI.createPost(postData);
            return response.data;
        } catch (error) {
            return rejectWithValue(error.response?.data?.message || 'Failed to create post');
        }
    }
);

// Updated user slice with async thunks
const userSlice = createSlice({
    name: 'user',
    initialState: {
        currentUser: null,
        loading: false,
        error: null
    },
    reducers: {
        logout: (state) => {
            state.currentUser = null;
            state.error = null;
            localStorage.removeItem('token');
        }
    },
    extraReducers: (builder) => {
        builder
            .addCase(loginUser.pending, (state) => {
                state.loading = true;
                state.error = null;
            })
            .addCase(loginUser.fulfilled, (state, action) => {
                state.loading = false;
                state.currentUser = action.payload;
            })
            .addCase(loginUser.rejected, (state, action) => {
                state.loading = false;
                state.error = action.payload;
            });
    }
});
```

---

## 🐻 Zustand State Management

### Basic Zustand Store

```javascript
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

// User Store
const useUserStore = create(
    devtools(
        persist(
            (set, get) => ({
                user: null,
                loading: false,
                error: null,
                
                login: async (credentials) => {
                    set({ loading: true, error: null });
                    try {
                        const response = await authAPI.login(credentials);
                        set({ 
                            user: response.data.user, 
                            loading: false 
                        });
                        localStorage.setItem('token', response.data.token);
                    } catch (error) {
                        set({ 
                            error: error.response?.data?.message || 'Login failed',
                            loading: false 
                        });
                    }
                },
                
                logout: () => {
                    set({ user: null, error: null });
                    localStorage.removeItem('token');
                },
                
                updateProfile: (profileData) => {
                    const currentUser = get().user;
                    set({ user: { ...currentUser, ...profileData } });
                }
            }),
            {
                name: 'user-store',
                partialize: (state) => ({ user: state.user })
            }
        )
    )
);

// Posts Store
const usePostsStore = create(
    devtools((set, get) => ({
        posts: [],
        loading: false,
        error: null,
        currentPost: null,
        
        fetchPosts: async (params) => {
            set({ loading: true, error: null });
            try {
                const response = await postsAPI.getPosts(params);
                set({ posts: response.data, loading: false });
            } catch (error) {
                set({ 
                    error: error.response?.data?.message || 'Failed to fetch posts',
                    loading: false 
                });
            }
        },
        
        addPost: (post) => {
            set((state) => ({ posts: [post, ...state.posts] }));
        },
        
        updatePost: (updatedPost) => {
            set((state) => ({
                posts: state.posts.map(post =>
                    post.id === updatedPost.id ? updatedPost : post
                )
            }));
        },
        
        deletePost: (postId) => {
            set((state) => ({
                posts: state.posts.filter(post => post.id !== postId)
            }));
        },
        
        setCurrentPost: (post) => {
            set({ currentPost: post });
        }
    }))
);

// Shopping Cart Store
const useCartStore = create(
    devtools(
        persist(
            (set, get) => ({
                items: [],
                
                addItem: (item) => {
                    const items = get().items;
                    const existingItem = items.find(i => i.id === item.id);
                    
                    if (existingItem) {
                        set({
                            items: items.map(i =>
                                i.id === item.id
                                    ? { ...i, quantity: i.quantity + 1 }
                                    : i
                            )
                        });
                    } else {
                        set({ items: [...items, { ...item, quantity: 1 }] });
                    }
                },
                
                removeItem: (id) => {
                    set((state) => ({
                        items: state.items.filter(item => item.id !== id)
                    }));
                },
                
                updateQuantity: (id, quantity) => {
                    set((state) => ({
                        items: state.items.map(item =>
                            item.id === id ? { ...item, quantity } : item
                        )
                    }));
                },
                
                clearCart: () => {
                    set({ items: [] });
                },
                
                getTotalPrice: () => {
                    const items = get().items;
                    return items.reduce((total, item) => total + (item.price * item.quantity), 0);
                },
                
                getTotalItems: () => {
                    const items = get().items;
                    return items.reduce((total, item) => total + item.quantity, 0);
                }
            }),
            {
                name: 'cart-store'
            }
        )
    )
);
```

---

## 💾 State Persistence

### Custom Persistence Hook

```javascript
import { useState, useEffect } from 'react';

const usePersistentState = (key, initialValue) => {
    const [state, setState] = useState(() => {
        try {
            const item = localStorage.getItem(key);
            return item ? JSON.parse(item) : initialValue;
        } catch (error) {
            console.error(`Error loading ${key} from localStorage:`, error);
            return initialValue;
        }
    });
    
    useEffect(() => {
        try {
            localStorage.setItem(key, JSON.stringify(state));
        } catch (error) {
            console.error(`Error saving ${key} to localStorage:`, error);
        }
    }, [key, state]);
    
    return [state, setState];
};

// Usage
const UserProfile = () => {
    const [preferences, setPreferences] = usePersistentState('user-preferences', {
        theme: 'light',
        language: 'en',
        notifications: true
    });
    
    const updatePreference = (key, value) => {
        setPreferences(prev => ({ ...prev, [key]: value }));
    };
    
    return (
        <div>
            <select 
                value={preferences.theme} 
                onChange={(e) => updatePreference('theme', e.target.value)}
            >
                <option value="light">Light</option>
                <option value="dark">Dark</option>
            </select>
        </div>
    );
};
```

---

## ⚡ Optimistic Updates

### Optimistic Update Pattern

```javascript
const useOptimisticUpdates = () => {
    const [posts, setPosts] = useState([]);
    const [loading, setLoading] = useState(false);
    
    const createPost = async (postData) => {
        // Optimistic update
        const optimisticPost = {
            id: Date.now(), // temporary ID
            ...postData,
            createdAt: new Date().toISOString(),
            isOptimistic: true
        };
        
        setPosts(prev => [optimisticPost, ...prev]);
        
        try {
            // Actual API call
            const response = await postsAPI.createPost(postData);
            
            // Replace optimistic post with real post
            setPosts(prev => prev.map(post =>
                post.id === optimisticPost.id ? response.data : post
            ));
        } catch (error) {
            // Rollback on error
            setPosts(prev => prev.filter(post => post.id !== optimisticPost.id));
            throw error;
        }
    };
    
    const updatePost = async (id, updates) => {
        // Store original post for rollback
        const originalPost = posts.find(post => post.id === id);
        
        // Optimistic update
        setPosts(prev => prev.map(post =>
            post.id === id ? { ...post, ...updates } : post
        ));
        
        try {
            const response = await postsAPI.updatePost(id, updates);
            setPosts(prev => prev.map(post =>
                post.id === id ? response.data : post
            ));
        } catch (error) {
            // Rollback on error
            setPosts(prev => prev.map(post =>
                post.id === id ? originalPost : post
            ));
            throw error;
        }
    };
    
    return { posts, createPost, updatePost };
};
```

---

## 📊 State Normalization

### Normalized State Structure

```javascript
// Normalized state structure
const useNormalizedState = () => {
    const [state, setState] = useState({
        users: {
            byId: {},
            allIds: []
        },
        posts: {
            byId: {},
            allIds: []
        },
        comments: {
            byId: {},
            allIds: []
        }
    });
    
    const addUser = (user) => {
        setState(prev => ({
            ...prev,
            users: {
                byId: { ...prev.users.byId, [user.id]: user },
                allIds: prev.users.allIds.includes(user.id) 
                    ? prev.users.allIds 
                    : [...prev.users.allIds, user.id]
            }
        }));
    };
    
    const addPost = (post) => {
        setState(prev => ({
            ...prev,
            posts: {
                byId: { ...prev.posts.byId, [post.id]: post },
                allIds: prev.posts.allIds.includes(post.id)
                    ? prev.posts.allIds
                    : [...prev.posts.allIds, post.id]
            }
        }));
    };
    
    const getPostWithAuthor = (postId) => {
        const post = state.posts.byId[postId];
        const author = state.users.byId[post?.authorId];
        return post ? { ...post, author } : null;
    };
    
    const getAllPostsWithAuthors = () => {
        return state.posts.allIds.map(id => getPostWithAuthor(id));
    };
    
    return {
        state,
        addUser,
        addPost,
        getPostWithAuthor,
        getAllPostsWithAuthors
    };
};
```

This intermediate state management guide covers advanced Context API patterns, Redux Toolkit, Zustand, state persistence, optimistic updates, and normalization techniques essential for building scalable React applications.
