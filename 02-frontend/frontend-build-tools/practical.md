
# 🔧 Intermediate Frontend Build Tools

## Table of Contents
- [Advanced Webpack Configuration](#advanced-webpack-configuration)
- [Custom Loaders and Plugins](#custom-loaders-and-plugins)
- [Code Splitting Strategies](#code-splitting-strategies)
- [Performance Optimization](#performance-optimization)
- [Environment Configuration](#environment-configuration)
- [Monorepo Build Setup](#monorepo-build-setup)
- [Asset Management](#asset-management)
- [Testing Integration](#testing-integration)
- [CI/CD Integration](#cicd-integration)

---

## ⚙️ Advanced Webpack Configuration

### Multi-Entry Configuration

```javascript
// webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    // Multiple entry points
    entry: {
        main: './src/index.js',
        admin: './src/admin/index.js',
        vendor: ['react', 'react-dom', 'lodash']
    },
    
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].[contenthash].js',
        chunkFilename: '[name].[contenthash].chunk.js',
        publicPath: '/',
        clean: true
    },
    
    optimization: {
        splitChunks: {
            chunks: 'all',
            cacheGroups: {
                default: {
                    minChunks: 2,
                    priority: -20,
                    reuseExistingChunk: true
                },
                vendor: {
                    test: /[\\/]node_modules[\\/]/,
                    name: 'vendors',
                    priority: -10,
                    chunks: 'all'
                },
                common: {
                    name: 'common',
                    minChunks: 2,
                    priority: -5,
                    reuseExistingChunk: true
                }
            }
        },
        runtimeChunk: {
            name: entrypoint => `runtime-${entrypoint.name}`
        }
    },
    
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html',
            filename: 'index.html',
            chunks: ['runtime-main', 'vendors', 'common', 'main']
        }),
        new HtmlWebpackPlugin({
            template: './src/admin/index.html',
            filename: 'admin.html',
            chunks: ['runtime-admin', 'vendors', 'common', 'admin']
        })
    ]
};
```

### Dynamic Configuration

```javascript
// webpack.config.js
const path = require('path');

module.exports = (env, argv) => {
    const isProduction = argv.mode === 'production';
    const isDevelopment = !isProduction;
    
    return {
        mode: argv.mode,
        devtool: isProduction ? 'source-map' : 'eval-source-map',
        
        entry: './src/index.js',
        
        output: {
            path: path.resolve(__dirname, 'dist'),
            filename: isProduction 
                ? '[name].[contenthash].js' 
                : '[name].js',
            publicPath: env.PUBLIC_PATH || '/'
        },
        
        module: {
            rules: [
                {
                    test: /\.js$/,
                    exclude: /node_modules/,
                    use: {
                        loader: 'babel-loader',
                        options: {
                            presets: ['@babel/preset-env'],
                            plugins: [
                                ...(isDevelopment ? ['react-refresh/babel'] : [])
                            ]
                        }
                    }
                },
                {
                    test: /\.css$/,
                    use: [
                        isProduction ? MiniCssExtractPlugin.loader : 'style-loader',
                        {
                            loader: 'css-loader',
                            options: {
                                modules: {
                                    localIdentName: isProduction 
                                        ? '[hash:base64]' 
                                        : '[name]__[local]__[hash:base64:5]'
                                }
                            }
                        }
                    ]
                }
            ]
        },
        
        plugins: [
            ...(isDevelopment ? [new ReactRefreshWebpackPlugin()] : []),
            ...(isProduction ? [
                new MiniCssExtractPlugin({
                    filename: '[name].[contenthash].css'
                })
            ] : [])
        ],
        
        devServer: isDevelopment ? {
            hot: true,
            port: env.PORT || 3000,
            historyApiFallback: true
        } : undefined
    };
};
```

---

## 🔌 Custom Loaders and Plugins

### Custom Loader

```javascript
// loaders/markdown-loader.js
const marked = require('marked');

module.exports = function(source) {
    // This is a loader that converts markdown to HTML
    const html = marked(source);
    
    // Return a JavaScript module that exports the HTML
    return `export default ${JSON.stringify(html)}`;
};

// Usage in webpack.config.js
module.exports = {
    module: {
        rules: [
            {
                test: /\.md$/,
                use: [
                    'html-loader',
                    path.resolve(__dirname, 'loaders/markdown-loader.js')
                ]
            }
        ]
    }
};

// Advanced loader with options
// loaders/svg-loader.js
const loaderUtils = require('loader-utils');

module.exports = function(source) {
    const options = loaderUtils.getOptions(this) || {};
    const { optimize = true, className = '' } = options;
    
    let svg = source;
    
    if (optimize) {
        // Optimize SVG (remove unnecessary attributes, etc.)
        svg = svg.replace(/\s+/g, ' ').trim();
    }
    
    if (className) {
        svg = svg.replace('<svg', `<svg class="${className}"`);
    }
    
    return `export default ${JSON.stringify(svg)}`;
};

// Usage
{
    test: /\.svg$/,
    use: [
        {
            loader: path.resolve(__dirname, 'loaders/svg-loader.js'),
            options: {
                optimize: true,
                className: 'svg-icon'
            }
        }
    ]
}
```

### Custom Plugin

```javascript
// plugins/build-info-plugin.js
class BuildInfoPlugin {
    constructor(options = {}) {
        this.options = options;
    }
    
    apply(compiler) {
        const pluginName = BuildInfoPlugin.name;
        
        // Hook into the compilation process
        compiler.hooks.compilation.tap(pluginName, (compilation) => {
            // Hook into the process assets phase
            compilation.hooks.processAssets.tap(
                {
                    name: pluginName,
                    stage: compilation.PROCESS_ASSETS_STAGE_SUMMARIZE
                },
                () => {
                    const buildInfo = {
                        buildTime: new Date().toISOString(),
                        version: require('../package.json').version,
                        environment: process.env.NODE_ENV,
                        assets: Object.keys(compilation.assets)
                    };
                    
                    // Add build info as an asset
                    compilation.emitAsset(
                        'build-info.json',
                        new compilation.compiler.webpack.sources.RawSource(
                            JSON.stringify(buildInfo, null, 2)
                        )
                    );
                }
            );
        });
        
        // Hook into done phase
        compiler.hooks.done.tap(pluginName, (stats) => {
            const { errors, warnings } = stats.compilation;
            
            console.log('\n📊 Build Summary:');
            console.log(`⏱️  Time: ${stats.endTime - stats.startTime}ms`);
            console.log(`📁 Assets: ${Object.keys(stats.compilation.assets).length}`);
            console.log(`❌ Errors: ${errors.length}`);
            console.log(`⚠️  Warnings: ${warnings.length}`);
        });
    }
}

module.exports = BuildInfoPlugin;

// Usage
const BuildInfoPlugin = require('./plugins/build-info-plugin');

module.exports = {
    plugins: [
        new BuildInfoPlugin({
            outputPath: 'meta'
        })
    ]
};
```

---

## 📦 Code Splitting Strategies

### Dynamic Imports

```javascript
// Route-based code splitting
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

// Lazy load components
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => 
    import('./pages/Dashboard').then(module => ({
        default: module.Dashboard
    }))
);

// Preload critical routes
const AdminPanel = lazy(() => {
    const componentImport = import('./pages/AdminPanel');
    
    // Preload dependencies
    import('./utils/adminHelpers');
    import('./components/AdminChart');
    
    return componentImport;
});

function App() {
    return (
        <Suspense fallback={<div className="loading">Loading...</div>}>
            <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/about" element={<About />} />
                <Route path="/dashboard" element={<Dashboard />} />
                <Route path="/admin" element={<AdminPanel />} />
            </Routes>
        </Suspense>
    );
}

// Feature-based splitting
const FeatureA = lazy(() =>
    import(
        /* webpackChunkName: "feature-a" */
        /* webpackPrefetch: true */
        './features/FeatureA'
    )
);

const FeatureB = lazy(() =>
    import(
        /* webpackChunkName: "feature-b" */
        /* webpackPreload: true */
        './features/FeatureB'
    )
);
```

### Advanced Splitting Configuration

```javascript
// webpack.config.js
module.exports = {
    optimization: {
        splitChunks: {
            chunks: 'all',
            minSize: 20000,
            maxSize: 244000,
            cacheGroups: {
                // Vendor chunks
                vendor: {
                    test: /[\\/]node_modules[\\/]/,
                    name: 'vendors',
                    chunks: 'all',
                    priority: 10
                },
                
                // React ecosystem
                react: {
                    test: /[\\/]node_modules[\\/](react|react-dom|react-router)[\\/]/,
                    name: 'react',
                    chunks: 'all',
                    priority: 20
                },
                
                // UI libraries
                ui: {
                    test: /[\\/]node_modules[\\/](@mui|antd|react-bootstrap)[\\/]/,
                    name: 'ui',
                    chunks: 'all',
                    priority: 15
                },
                
                // Common code
                common: {
                    minChunks: 2,
                    name: 'common',
                    chunks: 'all',
                    priority: 5,
                    reuseExistingChunk: true
                },
                
                // Styles
                styles: {
                    test: /\.css$/,
                    name: 'styles',
                    chunks: 'all',
                    enforce: true
                }
            }
        }
    }
};

// Dynamic chunk loading utility
class ChunkLoader {
    static loadedChunks = new Set();
    
    static async loadChunk(chunkName) {
        if (this.loadedChunks.has(chunkName)) {
            return;
        }
        
        try {
            await import(
                /* webpackChunkName: "[request]" */
                `./chunks/${chunkName}`
            );
            this.loadedChunks.add(chunkName);
        } catch (error) {
            console.error(`Failed to load chunk: ${chunkName}`, error);
        }
    }
    
    static preloadChunks(chunkNames) {
        chunkNames.forEach(chunkName => {
            const link = document.createElement('link');
            link.rel = 'prefetch';
            link.href = `./chunks/${chunkName}.js`;
            document.head.appendChild(link);
        });
    }
}
```

---

## 🚀 Performance Optimization

### Bundle Analysis and Optimization

```javascript
// webpack-analyzer.config.js
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
const SpeedMeasurePlugin = require('speed-measure-webpack-plugin');
const DuplicatePackageCheckerPlugin = require('duplicate-package-checker-webpack-plugin');

const smp = new SpeedMeasurePlugin();

module.exports = smp.wrap({
    // Your webpack config
    plugins: [
        // Bundle size analysis
        new BundleAnalyzerPlugin({
            analyzerMode: 'static',
            openAnalyzer: false,
            reportFilename: 'bundle-report.html'
        }),
        
        // Check for duplicate packages
        new DuplicatePackageCheckerPlugin({
            verbose: true,
            emitError: true
        }),
        
        // Webpack Bundle Analyzer
        new (require('webpack-bundle-analyzer').BundleAnalyzerPlugin)({
            analyzerMode: process.env.ANALYZE ? 'server' : 'disabled'
        })
    ],
    
    optimization: {
        // Module concatenation
        concatenateModules: true,
        
        // Minimize bundles
        minimize: true,
        minimizer: [
            new (require('terser-webpack-plugin'))({
                terserOptions: {
                    compress: {
                        drop_console: true,
                        drop_debugger: true,
                        pure_funcs: ['console.log', 'console.info']
                    },
                    mangle: {
                        safari10: true
                    }
                },
                extractComments: false
            })
        ]
    }
});

// Performance budget
module.exports = {
    performance: {
        maxAssetSize: 250000,
        maxEntrypointSize: 250000,
        hints: 'warning',
        assetFilter: function(assetFilename) {
            return assetFilename.endsWith('.js');
        }
    }
};
```

### Build Caching

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
    cache: {
        type: 'filesystem',
        cacheDirectory: path.resolve(__dirname, '.webpack-cache'),
        buildDependencies: {
            config: [__filename]
        }
    },
    
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /node_modules/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        cacheDirectory: true,
                        cacheCompression: false
                    }
                }
            }
        ]
    },
    
    optimization: {
        moduleIds: 'deterministic',
        chunkIds: 'deterministic'
    }
};

// Persistent caching for CI/CD
// .github/workflows/build.yml
- name: Cache Webpack
  uses: actions/cache@v3
  with:
    path: .webpack-cache
    key: webpack-${{ hashFiles('webpack.config.js', 'package-lock.json') }}
    restore-keys: |
      webpack-
```

---

## 🌍 Environment Configuration

### Environment-Specific Builds

```javascript
// config/webpack.common.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, '../dist'),
        clean: true
    },
    
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html'
        })
    ],
    
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /node_modules/,
                use: 'babel-loader'
            }
        ]
    }
};

// config/webpack.dev.js
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');
const ReactRefreshWebpackPlugin = require('@pmmmwh/react-refresh-webpack-plugin');

module.exports = merge(common, {
    mode: 'development',
    devtool: 'eval-source-map',
    
    output: {
        filename: '[name].js'
    },
    
    devServer: {
        hot: true,
        port: 3000,
        historyApiFallback: true,
        proxy: {
            '/api': 'http://localhost:5000'
        }
    },
    
    plugins: [
        new ReactRefreshWebpackPlugin()
    ]
});

// config/webpack.prod.js
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = merge(common, {
    mode: 'production',
    devtool: 'source-map',
    
    output: {
        filename: '[name].[contenthash].js'
    },
    
    plugins: [
        new MiniCssExtractPlugin({
            filename: '[name].[contenthash].css'
        })
    ],
    
    optimization: {
        splitChunks: {
            chunks: 'all'
        }
    }
});

// Environment variables
// .env.development
REACT_APP_API_URL=http://localhost:5000/api
REACT_APP_ANALYTICS_ID=dev-analytics
REACT_APP_LOG_LEVEL=debug

// .env.production
REACT_APP_API_URL=https://api.myapp.com
REACT_APP_ANALYTICS_ID=prod-analytics
REACT_APP_LOG_LEVEL=error

// webpack.config.js
const webpack = require('webpack');
const dotenv = require('dotenv');

module.exports = (env, argv) => {
    const currentPath = path.join(__dirname);
    const basePath = currentPath + '/.env';
    const envPath = basePath + '.' + argv.mode;
    
    const fileEnv = dotenv.config({ path: envPath }).parsed;
    const envKeys = Object.keys(fileEnv).reduce((prev, next) => {
        prev[`process.env.${next}`] = JSON.stringify(fileEnv[next]);
        return prev;
    }, {});
    
    return {
        plugins: [
            new webpack.DefinePlugin(envKeys)
        ]
    };
};
```

---

## 🏗️ Monorepo Build Setup

### Lerna + Webpack Configuration

```javascript
// packages/shared/webpack.config.js
module.exports = {
    mode: 'production',
    entry: './src/index.js',
    
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'index.js',
        library: '@myapp/shared',
        libraryTarget: 'umd'
    },
    
    externals: {
        react: 'react',
        'react-dom': 'react-dom'
    }
};

// packages/web-app/webpack.config.js
const path = require('path');

module.exports = {
    resolve: {
        alias: {
            '@shared': path.resolve(__dirname, '../shared/src'),
            '@components': path.resolve(__dirname, '../components/src')
        }
    },
    
    module: {
        rules: [
            {
                test: /\.js$/,
                include: [
                    path.resolve(__dirname, 'src'),
                    path.resolve(__dirname, '../shared/src'),
                    path.resolve(__dirname, '../components/src')
                ],
                use: 'babel-loader'
            }
        ]
    }
};

// Root package.json
{
    "scripts": {
        "build:shared": "lerna run build --scope=@myapp/shared",
        "build:components": "lerna run build --scope=@myapp/components",
        "build:web": "lerna run build --scope=@myapp/web",
        "build": "npm run build:shared && npm run build:components && npm run build:web",
        "dev": "lerna run dev --parallel"
    }
}
```

### Nx Workspace Configuration

```javascript
// nx.json
{
    "version": 2,
    "projects": {
        "web-app": "apps/web-app",
        "mobile-app": "apps/mobile-app",
        "shared-ui": "libs/shared-ui",
        "shared-utils": "libs/shared-utils"
    },
    "targetDefaults": {
        "build": {
            "dependsOn": ["^build"]
        }
    }
}

// apps/web-app/project.json
{
    "targets": {
        "build": {
            "executor": "@nrwl/webpack:webpack",
            "options": {
                "outputPath": "dist/apps/web-app",
                "index": "apps/web-app/src/index.html",
                "main": "apps/web-app/src/main.tsx",
                "polyfills": "apps/web-app/src/polyfills.ts",
                "tsConfig": "apps/web-app/tsconfig.app.json",
                "assets": ["apps/web-app/src/favicon.ico", "apps/web-app/src/assets"],
                "styles": ["apps/web-app/src/styles.css"],
                "scripts": [],
                "webpackConfig": "apps/web-app/webpack.config.js"
            }
        },
        "serve": {
            "executor": "@nrwl/webpack:dev-server",
            "options": {
                "buildTarget": "web-app:build",
                "port": 3000
            }
        }
    }
}
```

This intermediate build tools guide covers advanced webpack configurations, custom loaders and plugins, code splitting strategies, performance optimization, environment management, and monorepo setups essential for complex frontend projects.
