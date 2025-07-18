
# 🔧 Frontend Build Tools Basics

## Table of Contents
- [Introduction to Build Tools](#introduction-to-build-tools)
- [Webpack Fundamentals](#webpack-fundamentals)
- [Vite Build Tool](#vite-build-tool)
- [Package Managers](#package-managers)
- [Task Runners](#task-runners)
- [Code Bundling](#code-bundling)
- [Asset Optimization](#asset-optimization)
- [Development Server](#development-server)
- [Build Configuration](#build-configuration)

---

## 🚀 Introduction to Build Tools

Build tools are essential for modern web development, automating tasks like bundling, transpilation, optimization, and deployment.

### Why Use Build Tools?
- **Module Bundling**: Combine multiple files into optimized bundles
- **Code Transformation**: Transpile ES6+, TypeScript, JSX
- **Asset Optimization**: Minify CSS/JS, optimize images
- **Development Experience**: Hot reload, source maps
- **Performance**: Code splitting, tree shaking

### Popular Build Tools:
- **Webpack**: Powerful, configurable bundler
- **Vite**: Fast, modern build tool
- **Parcel**: Zero-configuration bundler
- **Rollup**: Optimized for libraries
- **esbuild**: Extremely fast bundler

---

## 📦 Webpack Fundamentals

### Basic Webpack Setup

```javascript
// webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
    // Entry point
    entry: './src/index.js',
    
    // Output configuration
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].[contenthash].js',
        clean: true
    },
    
    // Module rules for different file types
    module: {
        rules: [
            // JavaScript/JSX
            {
                test: /\.(js|jsx)$/,
                exclude: /node_modules/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: [
                            '@babel/preset-env',
                            '@babel/preset-react'
                        ]
                    }
                }
            },
            
            // CSS/SCSS
            {
                test: /\.(css|scss)$/,
                use: [
                    process.env.NODE_ENV === 'production' 
                        ? MiniCssExtractPlugin.loader 
                        : 'style-loader',
                    'css-loader',
                    'sass-loader'
                ]
            },
            
            // Images
            {
                test: /\.(png|jpg|jpeg|gif|svg)$/,
                type: 'asset/resource',
                generator: {
                    filename: 'images/[name].[hash][ext]'
                }
            },
            
            // Fonts
            {
                test: /\.(woff|woff2|eot|ttf|otf)$/,
                type: 'asset/resource',
                generator: {
                    filename: 'fonts/[name].[hash][ext]'
                }
            }
        ]
    },
    
    // Plugins
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html',
            minify: process.env.NODE_ENV === 'production'
        }),
        
        new MiniCssExtractPlugin({
            filename: '[name].[contenthash].css'
        })
    ],
    
    // Development server
    devServer: {
        contentBase: path.join(__dirname, 'dist'),
        port: 3000,
        hot: true,
        open: true
    },
    
    // Optimization
    optimization: {
        splitChunks: {
            chunks: 'all',
            cacheGroups: {
                vendor: {
                    test: /[\\/]node_modules[\\/]/,
                    name: 'vendors',
                    chunks: 'all'
                }
            }
        }
    },
    
    // Resolve extensions
    resolve: {
        extensions: ['.js', '.jsx', '.ts', '.tsx'],
        alias: {
            '@': path.resolve(__dirname, 'src'),
            '@components': path.resolve(__dirname, 'src/components'),
            '@utils': path.resolve(__dirname, 'src/utils')
        }
    }
};
```

### Advanced Webpack Configuration

```javascript
// webpack.prod.js
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');
const TerserPlugin = require('terser-webpack-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');

module.exports = merge(common, {
    mode: 'production',
    
    optimization: {
        minimize: true,
        minimizer: [
            new TerserPlugin({
                terserOptions: {
                    compress: {
                        drop_console: true,
                        drop_debugger: true
                    }
                }
            }),
            new CssMinimizerPlugin()
        ],
        
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
                react: {
                    test: /[\\/]node_modules[\\/](react|react-dom)[\\/]/,
                    name: 'react',
                    chunks: 'all'
                }
            }
        }
    },
    
    plugins: [
        ...(process.env.ANALYZE_BUNDLE && [
            new BundleAnalyzerPlugin({
                analyzerMode: 'static',
                openAnalyzer: false
            })
        ])
    ]
});

// webpack.dev.js
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
    mode: 'development',
    devtool: 'eval-source-map',
    
    devServer: {
        hot: true,
        port: 3000,
        historyApiFallback: true,
        proxy: {
            '/api': {
                target: 'http://localhost:5000',
                changeOrigin: true
            }
        }
    }
});
```

---

## ⚡ Vite Build Tool

### Vite Configuration

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { resolve } from 'path';

export default defineConfig({
    plugins: [react()],
    
    // Development server
    server: {
        port: 3000,
        hot: true,
        proxy: {
            '/api': {
                target: 'http://localhost:5000',
                changeOrigin: true
            }
        }
    },
    
    // Build configuration
    build: {
        outDir: 'dist',
        sourcemap: true,
        minify: 'terser',
        
        rollupOptions: {
            output: {
                manualChunks: {
                    vendor: ['react', 'react-dom'],
                    router: ['react-router-dom'],
                    ui: ['@mui/material', '@mui/icons-material']
                }
            }
        },
        
        // Asset handling
        assetsDir: 'assets',
        
        // Terser options
        terserOptions: {
            compress: {
                drop_console: true,
                drop_debugger: true
            }
        }
    },
    
    // Path resolution
    resolve: {
        alias: {
            '@': resolve(__dirname, 'src'),
            '@components': resolve(__dirname, 'src/components'),
            '@hooks': resolve(__dirname, 'src/hooks'),
            '@utils': resolve(__dirname, 'src/utils'),
            '@styles': resolve(__dirname, 'src/styles')
        }
    },
    
    // CSS configuration
    css: {
        preprocessorOptions: {
            scss: {
                additionalData: `@import "@/styles/variables.scss";`
            }
        },
        modules: {
            localsConvention: 'camelCase'
        }
    },
    
    // Environment variables
    define: {
        __APP_VERSION__: JSON.stringify(process.env.npm_package_version)
    }
});
```

### Vite Plugins

```javascript
// Custom Vite plugin
function customPlugin() {
    return {
        name: 'custom-plugin',
        
        buildStart() {
            console.log('Build started');
        },
        
        transform(code, id) {
            if (id.endsWith('.special')) {
                return `export default ${JSON.stringify(code)}`;
            }
        },
        
        generateBundle(options, bundle) {
            // Modify bundle before writing
            Object.keys(bundle).forEach(fileName => {
                const file = bundle[fileName];
                if (file.type === 'chunk') {
                    file.code = file.code.replace(/console\.log/g, '// console.log');
                }
            });
        }
    };
}

// vite.config.js with plugins
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { resolve } from 'path';
import legacy from '@vitejs/plugin-legacy';
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
    plugins: [
        react(),
        
        // Legacy browser support
        legacy({
            targets: ['defaults', 'not IE 11']
        }),
        
        // Bundle analyzer
        visualizer({
            filename: 'dist/stats.html',
            open: true,
            gzipSize: true
        }),
        
        // Custom plugin
        customPlugin()
    ]
});
```

---

## 📱 Package Managers

### npm Scripts

```json
{
    "name": "my-app",
    "version": "1.0.0",
    "scripts": {
        "dev": "vite",
        "build": "vite build",
        "preview": "vite preview",
        "test": "jest",
        "test:watch": "jest --watch",
        "test:coverage": "jest --coverage",
        "lint": "eslint src --ext .js,.jsx,.ts,.tsx",
        "lint:fix": "eslint src --ext .js,.jsx,.ts,.tsx --fix",
        "format": "prettier --write \"src/**/*.{js,jsx,ts,tsx,css,scss,md}\"",
        "type-check": "tsc --noEmit",
        "analyze": "npm run build && npx webpack-bundle-analyzer dist/stats.json",
        "clean": "rimraf dist",
        "start": "npm run dev",
        "prebuild": "npm run clean",
        "postbuild": "npm run analyze"
    },
    "dependencies": {
        "react": "^18.2.0",
        "react-dom": "^18.2.0"
    },
    "devDependencies": {
        "@vitejs/plugin-react": "^4.0.0",
        "vite": "^4.3.0",
        "eslint": "^8.0.0",
        "prettier": "^2.8.0",
        "jest": "^29.0.0",
        "typescript": "^5.0.0"
    }
}
```

### Yarn Workspaces

```json
// package.json (root)
{
    "name": "my-monorepo",
    "private": true,
    "workspaces": [
        "packages/*",
        "apps/*"
    ],
    "scripts": {
        "build": "yarn workspaces run build",
        "test": "yarn workspaces run test",
        "dev": "yarn workspace @myapp/web dev"
    },
    "devDependencies": {
        "lerna": "^6.0.0"
    }
}

// packages/ui/package.json
{
    "name": "@myapp/ui",
    "version": "1.0.0",
    "main": "dist/index.js",
    "scripts": {
        "build": "tsc",
        "dev": "tsc --watch"
    },
    "dependencies": {
        "react": "^18.2.0"
    }
}

// apps/web/package.json
{
    "name": "@myapp/web",
    "version": "1.0.0",
    "scripts": {
        "dev": "vite",
        "build": "vite build"
    },
    "dependencies": {
        "@myapp/ui": "workspace:*",
        "react": "^18.2.0"
    }
}
```

---

## 🎯 Task Runners

### Gulp Configuration

```javascript
// gulpfile.js
const gulp = require('gulp');
const sass = require('gulp-sass')(require('sass'));
const autoprefixer = require('gulp-autoprefixer');
const cssnano = require('gulp-cssnano');
const uglify = require('gulp-uglify');
const imagemin = require('gulp-imagemin');
const concat = require('gulp-concat');
const sourcemaps = require('gulp-sourcemaps');
const browsersync = require('browser-sync').create();

// Paths
const paths = {
    styles: {
        src: 'src/scss/**/*.scss',
        dest: 'dist/css/'
    },
    scripts: {
        src: 'src/js/**/*.js',
        dest: 'dist/js/'
    },
    images: {
        src: 'src/images/**/*',
        dest: 'dist/images/'
    },
    html: {
        src: 'src/**/*.html',
        dest: 'dist/'
    }
};

// Styles task
function styles() {
    return gulp
        .src(paths.styles.src)
        .pipe(sourcemaps.init())
        .pipe(sass().on('error', sass.logError))
        .pipe(autoprefixer({
            cascade: false
        }))
        .pipe(cssnano())
        .pipe(sourcemaps.write('.'))
        .pipe(gulp.dest(paths.styles.dest))
        .pipe(browsersync.stream());
}

// Scripts task
function scripts() {
    return gulp
        .src(paths.scripts.src)
        .pipe(sourcemaps.init())
        .pipe(concat('main.js'))
        .pipe(uglify())
        .pipe(sourcemaps.write('.'))
        .pipe(gulp.dest(paths.scripts.dest))
        .pipe(browsersync.stream());
}

// Images task
function images() {
    return gulp
        .src(paths.images.src)
        .pipe(imagemin([
            imagemin.gifsicle({ interlaced: true }),
            imagemin.mozjpeg({ quality: 75, progressive: true }),
            imagemin.optipng({ optimizationLevel: 5 }),
            imagemin.svgo({
                plugins: [
                    { removeViewBox: false },
                    { cleanupIDs: false }
                ]
            })
        ]))
        .pipe(gulp.dest(paths.images.dest));
}

// HTML task
function html() {
    return gulp
        .src(paths.html.src)
        .pipe(gulp.dest(paths.html.dest))
        .pipe(browsersync.stream());
}

// Watch task
function watch() {
    browsersync.init({
        server: {
            baseDir: './dist'
        }
    });
    
    gulp.watch(paths.styles.src, styles);
    gulp.watch(paths.scripts.src, scripts);
    gulp.watch(paths.images.src, images);
    gulp.watch(paths.html.src, html);
}

// Build task
const build = gulp.series(
    gulp.parallel(styles, scripts, images, html)
);

// Export tasks
exports.styles = styles;
exports.scripts = scripts;
exports.images = images;
exports.html = html;
exports.watch = watch;
exports.build = build;
exports.default = build;
```

---

## 📦 Code Bundling Strategies

### Module Federation (Webpack 5)

```javascript
// Host application webpack config
const ModuleFederationPlugin = require('@module-federation/webpack');

module.exports = {
    mode: 'development',
    devServer: {
        port: 3000
    },
    plugins: [
        new ModuleFederationPlugin({
            name: 'host',
            remotes: {
                mfe1: 'mfe1@http://localhost:3001/remoteEntry.js',
                mfe2: 'mfe2@http://localhost:3002/remoteEntry.js'
            }
        })
    ]
};

// Micro-frontend webpack config
module.exports = {
    mode: 'development',
    devServer: {
        port: 3001
    },
    plugins: [
        new ModuleFederationPlugin({
            name: 'mfe1',
            filename: 'remoteEntry.js',
            exposes: {
                './Button': './src/Button',
                './Header': './src/Header'
            },
            shared: {
                react: { singleton: true },
                'react-dom': { singleton: true }
            }
        })
    ]
};

// Usage in host application
import React, { Suspense } from 'react';

const RemoteButton = React.lazy(() => import('mfe1/Button'));

function App() {
    return (
        <div>
            <h1>Host Application</h1>
            <Suspense fallback={<div>Loading...</div>}>
                <RemoteButton />
            </Suspense>
        </div>
    );
}
```

### Tree Shaking Configuration

```javascript
// webpack.config.js
module.exports = {
    mode: 'production',
    optimization: {
        usedExports: true,
        sideEffects: false
    },
    resolve: {
        // Enable tree shaking for ES modules
        mainFields: ['es2015', 'module', 'main']
    }
};

// package.json
{
    "name": "my-library",
    "main": "dist/index.js",
    "module": "dist/index.esm.js",
    "sideEffects": false,
    "exports": {
        ".": {
            "import": "./dist/index.esm.js",
            "require": "./dist/index.js"
        },
        "./components": {
            "import": "./dist/components.esm.js",
            "require": "./dist/components.js"
        }
    }
}

// Optimized imports
// ❌ Imports entire library
import { Button } from 'my-ui-library';

// ✅ Tree-shakeable import
import Button from 'my-ui-library/components/Button';

// ✅ With babel-plugin-import
import { Button } from 'my-ui-library'; // Transforms to specific import
```

---

## 🔧 Development Tools Integration

### ESLint and Prettier Setup

```javascript
// .eslintrc.js
module.exports = {
    env: {
        browser: true,
        es2021: true,
        node: true
    },
    extends: [
        'eslint:recommended',
        '@typescript-eslint/recommended',
        'plugin:react/recommended',
        'plugin:react-hooks/recommended',
        'prettier'
    ],
    parser: '@typescript-eslint/parser',
    parserOptions: {
        ecmaFeatures: {
            jsx: true
        },
        ecmaVersion: 12,
        sourceType: 'module'
    },
    plugins: [
        'react',
        '@typescript-eslint',
        'import'
    ],
    rules: {
        'react/react-in-jsx-scope': 'off',
        'react/prop-types': 'off',
        '@typescript-eslint/no-unused-vars': 'error',
        'import/order': [
            'error',
            {
                groups: [
                    'builtin',
                    'external',
                    'internal',
                    'parent',
                    'sibling',
                    'index'
                ],
                'newlines-between': 'always'
            }
        ]
    },
    settings: {
        react: {
            version: 'detect'
        }
    }
};

// .prettierrc.js
module.exports = {
    semi: true,
    trailingComma: 'es5',
    singleQuote: true,
    printWidth: 80,
    tabWidth: 2,
    useTabs: false
};

// .editorconfig
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 2

[*.md]
trim_trailing_whitespace = false
```

This comprehensive guide covers the fundamentals of frontend build tools, providing practical examples for webpack, Vite, package managers, and development workflow optimization.
