
# 🔄 Advanced State Management

## Table of Contents
- [State Machines](#state-machines)
- [Recoil State Management](#recoil-state-management)
- [MobX State Management](#mobx-state-management)
- [Real-time State Synchronization](#real-time-state-synchronization)
- [State Middleware](#state-middleware)
- [Micro-frontends State](#micro-frontends-state)
- [State Testing Strategies](#state-testing-strategies)
- [Performance Optimization](#performance-optimization)
- [State Architecture Patterns](#state-architecture-patterns)

---

## 🎰 State Machines

### XState Implementation

```javascript
import { createMachine, interpret } from 'xstate';

// Authentication state machine
const authMachine = createMachine({
    id: 'auth',
    initial: 'idle',
    context: {
        user: null,
        error: null,
        attempts: 0
    },
    states: {
        idle: {
            on: {
                LOGIN: 'authenticating'
            }
        },
        authenticating: {
            invoke: {
                id: 'login',
                src: 'loginUser',
                onDone: {
                    target: 'authenticated',
                    actions: 'setUser'
                },
                onError: {
                    target: 'failed',
                    actions: 'setError'
                }
            }
        },
        authenticated: {
            on: {
                LOGOUT: 'idle',
                REFRESH_TOKEN: 'refreshing'
            },
            initial: 'active',
            states: {
                active: {
                    after: {
                        300000: 'idle' // Auto logout after 5 minutes
                    }
                },
                idle: {
                    on: {
                        ACTIVITY: 'active'
                    }
                }
            }
        },
        failed: {
            on: {
                RETRY: {
                    target: 'authenticating',
                    cond: 'canRetry'
                },
                RESET: 'idle'
            }
        },
        refreshing: {
            invoke: {
                id: 'refresh',
                src: 'refreshToken',
                onDone: 'authenticated',
                onError: 'idle'
            }
        }
    }
}, {
    actions: {
        setUser: (context, event) => ({
            ...context,
            user: event.data,
            error: null
        }),
        setError: (context, event) => ({
            ...context,
            error: event.data,
            attempts: context.attempts + 1
        })
    },
    guards: {
        canRetry: (context) => context.attempts < 3
    },
    services: {
        loginUser: async (context, event) => {
            const response = await authAPI.login(event.credentials);
            return response.data;
        },
        refreshToken: async (context) => {
            const response = await authAPI.refreshToken();
            return response.data;
        }
    }
});

// React hook for state machine
const useAuthMachine = () => {
    const [state, send] = useActor(authMachine);
    
    const login = (credentials) => {
        send('LOGIN', { credentials });
    };
    
    const logout = () => {
        send('LOGOUT');
    };
    
    const retry = () => {
        send('RETRY');
    };
    
    return {
        state: state.value,
        context: state.context,
        login,
        logout,
        retry,
        isAuthenticated: state.matches('authenticated'),
        isLoading: state.matches('authenticating'),
        hasError: state.matches('failed')
    };
};
```

### Complex State Machine

```javascript
// Shopping cart state machine
const cartMachine = createMachine({
    id: 'cart',
    initial: 'idle',
    context: {
        items: [],
        total: 0,
        shippingInfo: null,
        paymentInfo: null
    },
    states: {
        idle: {
            on: {
                ADD_ITEM: {
                    actions: 'addItem'
                },
                REMOVE_ITEM: {
                    actions: 'removeItem'
                },
                CHECKOUT: 'checkout'
            }
        },
        checkout: {
            initial: 'shipping',
            states: {
                shipping: {
                    on: {
                        SUBMIT_SHIPPING: {
                            target: 'payment',
                            actions: 'setShipping'
                        },
                        BACK: '#cart.idle'
                    }
                },
                payment: {
                    on: {
                        SUBMIT_PAYMENT: {
                            target: 'processing',
                            actions: 'setPayment'
                        },
                        BACK: 'shipping'
                    }
                },
                processing: {
                    invoke: {
                        id: 'processPayment',
                        src: 'processPayment',
                        onDone: 'success',
                        onError: 'error'
                    }
                },
                success: {
                    entry: 'clearCart',
                    on: {
                        CONTINUE_SHOPPING: '#cart.idle'
                    }
                },
                error: {
                    on: {
                        RETRY: 'payment',
                        CANCEL: '#cart.idle'
                    }
                }
            }
        }
    }
}, {
    actions: {
        addItem: (context, event) => ({
            ...context,
            items: [...context.items, event.item],
            total: context.total + event.item.price
        }),
        removeItem: (context, event) => ({
            ...context,
            items: context.items.filter(item => item.id !== event.id),
            total: context.total - event.price
        }),
        setShipping: (context, event) => ({
            ...context,
            shippingInfo: event.data
        }),
        setPayment: (context, event) => ({
            ...context,
            paymentInfo: event.data
        }),
        clearCart: (context) => ({
            ...context,
            items: [],
            total: 0,
            shippingInfo: null,
            paymentInfo: null
        })
    },
    services: {
        processPayment: async (context) => {
            return await paymentAPI.processPayment({
                items: context.items,
                total: context.total,
                shippingInfo: context.shippingInfo,
                paymentInfo: context.paymentInfo
            });
        }
    }
});
```

---

## ⚛️ Recoil State Management

### Recoil Setup

```javascript
import { RecoilRoot, atom, selector, useRecoilState, useRecoilValue } from 'recoil';

// Atoms (state)
const userState = atom({
    key: 'userState',
    default: null
});

const postsState = atom({
    key: 'postsState',
    default: []
});

const filterState = atom({
    key: 'filterState',
    default: {
        category: '',
        searchTerm: '',
        sortBy: 'createdAt'
    }
});

// Selectors (derived state)
const filteredPostsState = selector({
    key: 'filteredPostsState',
    get: ({ get }) => {
        const posts = get(postsState);
        const filter = get(filterState);
        
        return posts
            .filter(post => {
                const matchesCategory = !filter.category || post.category === filter.category;
                const matchesSearch = !filter.searchTerm || 
                    post.title.toLowerCase().includes(filter.searchTerm.toLowerCase()) ||
                    post.content.toLowerCase().includes(filter.searchTerm.toLowerCase());
                
                return matchesCategory && matchesSearch;
            })
            .sort((a, b) => {
                switch (filter.sortBy) {
                    case 'createdAt':
                        return new Date(b.createdAt) - new Date(a.createdAt);
                    case 'title':
                        return a.title.localeCompare(b.title);
                    case 'likes':
                        return b.likes - a.likes;
                    default:
                        return 0;
                }
            });
    }
});

const userStatsState = selector({
    key: 'userStatsState',
    get: ({ get }) => {
        const user = get(userState);
        const posts = get(postsState);
        
        if (!user) return null;
        
        const userPosts = posts.filter(post => post.authorId === user.id);
        const totalLikes = userPosts.reduce((sum, post) => sum + post.likes, 0);
        
        return {
            totalPosts: userPosts.length,
            totalLikes,
            averageLikes: userPosts.length > 0 ? totalLikes / userPosts.length : 0
        };
    }
});

// Async selectors
const userProfileState = selector({
    key: 'userProfileState',
    get: async ({ get }) => {
        const user = get(userState);
        if (!user) return null;
        
        const response = await userAPI.getProfile(user.id);
        return response.data;
    }
});

// Atom families (parameterized atoms)
const postState = atomFamily({
    key: 'postState',
    default: (id) => ({
        id,
        title: '',
        content: '',
        likes: 0,
        comments: []
    })
});

const commentState = atomFamily({
    key: 'commentState',
    default: (postId) => []
});

// Recoil hooks usage
const PostList = () => {
    const filteredPosts = useRecoilValue(filteredPostsState);
    const [filter, setFilter] = useRecoilState(filterState);
    
    const updateFilter = (updates) => {
        setFilter(prev => ({ ...prev, ...updates }));
    };
    
    return (
        <div>
            <input
                type="text"
                value={filter.searchTerm}
                onChange={(e) => updateFilter({ searchTerm: e.target.value })}
                placeholder="Search posts..."
            />
            
            <select
                value={filter.category}
                onChange={(e) => updateFilter({ category: e.target.value })}
            >
                <option value="">All Categories</option>
                <option value="tech">Technology</option>
                <option value="design">Design</option>
                <option value="business">Business</option>
            </select>
            
            {filteredPosts.map(post => (
                <div key={post.id}>
                    <h3>{post.title}</h3>
                    <p>{post.content}</p>
                    <span>{post.likes} likes</span>
                </div>
            ))}
        </div>
    );
};
```

---

## 🏪 MobX State Management

### MobX Store Implementation

```javascript
import { makeObservable, observable, action, computed, runInAction } from 'mobx';

class UserStore {
    user = null;
    loading = false;
    error = null;
    
    constructor() {
        makeObservable(this, {
            user: observable,
            loading: observable,
            error: observable,
            login: action,
            logout: action,
            updateProfile: action,
            isAuthenticated: computed
        });
    }
    
    get isAuthenticated() {
        return !!this.user;
    }
    
    async login(credentials) {
        this.loading = true;
        this.error = null;
        
        try {
            const response = await authAPI.login(credentials);
            
            runInAction(() => {
                this.user = response.data.user;
                this.loading = false;
            });
            
            localStorage.setItem('token', response.data.token);
        } catch (error) {
            runInAction(() => {
                this.error = error.response?.data?.message || 'Login failed';
                this.loading = false;
            });
        }
    }
    
    logout() {
        this.user = null;
        this.error = null;
        localStorage.removeItem('token');
    }
    
    updateProfile(updates) {
        this.user = { ...this.user, ...updates };
    }
}

class PostsStore {
    posts = [];
    loading = false;
    error = null;
    
    constructor() {
        makeObservable(this, {
            posts: observable,
            loading: observable,
            error: observable,
            fetchPosts: action,
            addPost: action,
            updatePost: action,
            deletePost: action,
            postsCount: computed,
            featuredPosts: computed
        });
    }
    
    get postsCount() {
        return this.posts.length;
    }
    
    get featuredPosts() {
        return this.posts.filter(post => post.featured);
    }
    
    async fetchPosts() {
        this.loading = true;
        this.error = null;
        
        try {
            const response = await postsAPI.getPosts();
            
            runInAction(() => {
                this.posts = response.data;
                this.loading = false;
            });
        } catch (error) {
            runInAction(() => {
                this.error = error.response?.data?.message || 'Failed to fetch posts';
                this.loading = false;
            });
        }
    }
    
    addPost(post) {
        this.posts.unshift(post);
    }
    
    updatePost(id, updates) {
        const index = this.posts.findIndex(post => post.id === id);
        if (index !== -1) {
            this.posts[index] = { ...this.posts[index], ...updates };
        }
    }
    
    deletePost(id) {
        this.posts = this.posts.filter(post => post.id !== id);
    }
}

// Root store
class RootStore {
    constructor() {
        this.userStore = new UserStore();
        this.postsStore = new PostsStore();
    }
}

// React integration
import { observer } from 'mobx-react-lite';

const PostList = observer(() => {
    const { postsStore } = useContext(RootStoreContext);
    
    useEffect(() => {
        postsStore.fetchPosts();
    }, [postsStore]);
    
    if (postsStore.loading) return <div>Loading...</div>;
    if (postsStore.error) return <div>Error: {postsStore.error}</div>;
    
    return (
        <div>
            <h2>Posts ({postsStore.postsCount})</h2>
            {postsStore.posts.map(post => (
                <div key={post.id}>
                    <h3>{post.title}</h3>
                    <p>{post.content}</p>
                </div>
            ))}
        </div>
    );
});
```

---

## 🔄 Real-time State Synchronization

### WebSocket State Sync

```javascript
import { useEffect, useRef } from 'react';
import { useImmer } from 'use-immer';

const useRealtimeState = (initialState, websocketUrl) => {
    const [state, updateState] = useImmer(initialState);
    const ws = useRef(null);
    const reconnectTimeoutRef = useRef(null);
    const reconnectAttempts = useRef(0);
    
    const connect = () => {
        ws.current = new WebSocket(websocketUrl);
        
        ws.current.onopen = () => {
            console.log('WebSocket connected');
            reconnectAttempts.current = 0;
        };
        
        ws.current.onmessage = (event) => {
            const data = JSON.parse(event.data);
            
            switch (data.type) {
                case 'STATE_UPDATE':
                    updateState(draft => {
                        Object.assign(draft, data.payload);
                    });
                    break;
                    
                case 'ITEM_ADDED':
                    updateState(draft => {
                        draft.items.push(data.payload);
                    });
                    break;
                    
                case 'ITEM_UPDATED':
                    updateState(draft => {
                        const index = draft.items.findIndex(item => item.id === data.payload.id);
                        if (index !== -1) {
                            draft.items[index] = data.payload;
                        }
                    });
                    break;
                    
                case 'ITEM_REMOVED':
                    updateState(draft => {
                        draft.items = draft.items.filter(item => item.id !== data.payload.id);
                    });
                    break;
            }
        };
        
        ws.current.onclose = () => {
            console.log('WebSocket disconnected');
            
            // Exponential backoff reconnection
            const timeout = Math.min(1000 * Math.pow(2, reconnectAttempts.current), 30000);
            reconnectTimeoutRef.current = setTimeout(() => {
                reconnectAttempts.current++;
                connect();
            }, timeout);
        };
        
        ws.current.onerror = (error) => {
            console.error('WebSocket error:', error);
        };
    };
    
    const sendUpdate = (type, payload) => {
        if (ws.current && ws.current.readyState === WebSocket.OPEN) {
            ws.current.send(JSON.stringify({ type, payload }));
        }
    };
    
    const optimisticUpdate = (type, payload, rollback) => {
        // Apply optimistic update
        updateState(draft => {
            switch (type) {
                case 'ADD_ITEM':
                    draft.items.push(payload);
                    break;
                case 'UPDATE_ITEM':
                    const index = draft.items.findIndex(item => item.id === payload.id);
                    if (index !== -1) {
                        draft.items[index] = payload;
                    }
                    break;
            }
        });
        
        // Send to server
        sendUpdate(type, payload);
        
        // Set timeout for rollback if no confirmation received
        setTimeout(() => {
            if (rollback) {
                updateState(rollback);
            }
        }, 5000);
    };
    
    useEffect(() => {
        connect();
        
        return () => {
            if (reconnectTimeoutRef.current) {
                clearTimeout(reconnectTimeoutRef.current);
            }
            if (ws.current) {
                ws.current.close();
            }
        };
    }, [websocketUrl]);
    
    return { state, updateState, sendUpdate, optimisticUpdate };
};

// Usage
const CollaborativeEditor = () => {
    const { state, updateState, optimisticUpdate } = useRealtimeState(
        { content: '', collaborators: [] },
        'ws://localhost:8080/editor'
    );
    
    const handleContentChange = (newContent) => {
        optimisticUpdate('UPDATE_CONTENT', { content: newContent });
    };
    
    return (
        <div>
            <textarea
                value={state.content}
                onChange={(e) => handleContentChange(e.target.value)}
            />
            <div>
                Collaborators: {state.collaborators.map(c => c.name).join(', ')}
            </div>
        </div>
    );
};
```

---

## 🔧 State Middleware

### Redux Middleware

```javascript
// Logger middleware
const loggerMiddleware = (store) => (next) => (action) => {
    console.group(action.type);
    console.info('dispatching', action);
    console.log('prev state', store.getState());
    
    let result = next(action);
    
    console.log('next state', store.getState());
    console.groupEnd();
    
    return result;
};

// Analytics middleware
const analyticsMiddleware = (store) => (next) => (action) => {
    // Track user actions
    if (action.type.startsWith('USER_')) {
        analytics.track('User Action', {
            action: action.type,
            timestamp: new Date().toISOString(),
            userId: store.getState().user?.id
        });
    }
    
    return next(action);
};

// Persistence middleware
const persistenceMiddleware = (store) => (next) => (action) => {
    const result = next(action);
    
    // Save specific state to localStorage
    const state = store.getState();
    if (action.type.startsWith('USER_')) {
        localStorage.setItem('user', JSON.stringify(state.user));
    }
    
    return result;
};

// Error handling middleware
const errorMiddleware = (store) => (next) => (action) => {
    try {
        return next(action);
    } catch (error) {
        console.error('Redux error:', error);
        
        // Send error to monitoring service
        errorReporting.captureException(error, {
            extra: {
                action,
                state: store.getState()
            }
        });
        
        // Dispatch error action
        store.dispatch({
            type: 'ERROR_OCCURRED',
            payload: {
                message: error.message,
                stack: error.stack
            }
        });
        
        throw error;
    }
};
```

---

## 🧪 State Testing Strategies

### Testing State Logic

```javascript
// Testing Redux actions and reducers
import { createStore } from 'redux';
import { userReducer, loginSuccess, loginFailure } from './userSlice';

describe('User Reducer', () => {
    test('should handle login success', () => {
        const initialState = { user: null, loading: false, error: null };
        const user = { id: 1, email: 'test@example.com' };
        
        const action = loginSuccess(user);
        const newState = userReducer(initialState, action);
        
        expect(newState.user).toEqual(user);
        expect(newState.loading).toBe(false);
        expect(newState.error).toBe(null);
    });
    
    test('should handle login failure', () => {
        const initialState = { user: null, loading: true, error: null };
        const error = 'Invalid credentials';
        
        const action = loginFailure(error);
        const newState = userReducer(initialState, action);
        
        expect(newState.user).toBe(null);
        expect(newState.loading).toBe(false);
        expect(newState.error).toBe(error);
    });
});

// Testing async thunks
import { loginUser } from './userSlice';
import { authAPI } from './api';

jest.mock('./api');

describe('Login Thunk', () => {
    test('should login user successfully', async () => {
        const user = { id: 1, email: 'test@example.com' };
        authAPI.login.mockResolvedValue({ data: { user, token: 'token123' } });
        
        const dispatch = jest.fn();
        const getState = jest.fn();
        
        await loginUser({ email: 'test@example.com', password: 'password' })(dispatch, getState);
        
        expect(dispatch).toHaveBeenCalledWith(expect.objectContaining({
            type: 'user/login/pending'
        }));
        
        expect(dispatch).toHaveBeenCalledWith(expect.objectContaining({
            type: 'user/login/fulfilled',
            payload: user
        }));
    });
});

// Testing Zustand stores
import { act, renderHook } from '@testing-library/react';
import { useUserStore } from './userStore';

describe('User Store', () => {
    test('should login user', async () => {
        const { result } = renderHook(() => useUserStore());
        
        authAPI.login.mockResolvedValue({
            data: { user: { id: 1, email: 'test@example.com' }, token: 'token123' }
        });
        
        await act(async () => {
            await result.current.login({ email: 'test@example.com', password: 'password' });
        });
        
        expect(result.current.user).toBeTruthy();
        expect(result.current.loading).toBe(false);
    });
});
```

This advanced state management guide covers state machines, Recoil, MobX, real-time synchronization, middleware patterns, and comprehensive testing strategies for building sophisticated state management solutions.
