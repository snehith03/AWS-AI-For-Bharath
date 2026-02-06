# System Design Document
## Retail Market Intelligence & Dynamic Pricing Copilot

**Version:** 1.0  
**Date:** February 6, 2026  
**Document Owner:** Engineering Architecture Team  
**Status:** Design Review  
**Classification:** Internal

---


---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [High-Level System Architecture](#2-high-level-system-architecture)
3. [Component-Level Design](#3-component-level-design)
4. [Data Flow Architecture](#4-data-flow-architecture)
5. [Database Design](#5-database-design)
6. [API Design](#6-api-design)
7. [ML Pipeline Design](#7-ml-pipeline-design)
8. [Deployment Architecture](#8-deployment-architecture)
9. [Security Architecture](#9-security-architecture)
10. [Scalability & Performance](#10-scalability--performance)
11. [Monitoring & Observability](#11-monitoring--observability)
12. [Disaster Recovery](#12-disaster-recovery)

---

## 1. Executive Summary

This document presents the technical design for the Retail Market Intelligence & Dynamic Pricing Copilot, a cloud-native SaaS platform that provides AI-powered analytics for small and medium retailers. The system employs a microservices-oriented architecture with three primary components:

- **Frontend**: React-based SPA for user interaction
- **Backend API**: Node.js/Express RESTful service for business logic
- **ML Service**: Python/Flask microservice for machine learning operations

The architecture prioritizes:
- **Modularity**: Independent deployment and scaling of components
- **Reliability**: 99.5% uptime with automated failover
- **Security**: End-to-end encryption, JWT authentication, RBAC
- **Performance**: Sub-500ms API response times, <3s ML predictions
- **Scalability**: Horizontal scaling to support 10K+ concurrent users


## 2. High-Level System Architecture

### 2.1 Architecture Overview

The system follows a **three-tier architecture** with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────────────────┐
│                          PRESENTATION LAYER                          │
│                                                                       │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    React SPA (Client)                         │  │
│  │  - Component-based UI    - State Management (Context/Query)  │  │
│  │  - Responsive Design     - Client-side Routing               │  │
│  └──────────────────────────────────────────────────────────────┘  │
└───────────────────────────────┬─────────────────────────────────────┘
                                │ HTTPS/REST
                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         APPLICATION LAYER                            │
│                                                                       │
│  ┌────────────────────────────────────────────────────────────┐    │
│  │              API Gateway / Load Balancer                    │    │
│  │  - Request Routing    - Rate Limiting    - SSL Termination │    │
│  └────────────────────────────────────────────────────────────┘    │
│                                                                       │
│  ┌─────────────────────────┐      ┌─────────────────────────┐      │
│  │   Node.js API Server    │      │   Python ML Service     │      │
│  │   (Express.js)          │◄────►│   (Flask)               │      │
│  │                         │ HTTP │                         │      │
│  │ - Authentication        │      │ - Demand Forecasting    │      │
│  │ - Business Logic        │      │ - Price Optimization    │      │
│  │ - Data Validation       │      │ - Customer Segmentation │      │
│  │ - API Orchestration     │      │ - Model Training        │      │
│  └─────────────────────────┘      └─────────────────────────┘      │
│              │                                    │                  │
└──────────────┼────────────────────────────────────┼──────────────────┘
               │                                    │
               ▼                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                           DATA LAYER                                 │
│                                                                       │
│  ┌─────────────────────────┐      ┌─────────────────────────┐      │
│  │   MongoDB (Primary)     │      │   Redis Cache           │      │
│  │   - User Data           │      │   - Session Storage     │      │
│  │   - Products            │      │   - API Response Cache  │      │
│  │   - Sales Transactions  │      │   - Rate Limit Counters │      │
│  │   - Analytics Data      │      └─────────────────────────┘      │
│  └─────────────────────────┘                                        │
│                                                                       │
│  ┌─────────────────────────┐      ┌─────────────────────────┐      │
│  │   S3 Object Storage     │      │   Model Storage         │      │
│  │   - CSV Imports         │      │   - Trained ML Models   │      │
│  │   - Generated Reports   │      │   - Model Metadata      │      │
│  │   - User Uploads        │      │   - Training Datasets   │      │
│  └─────────────────────────┘      └─────────────────────────┘      │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 Architecture Principles

**Separation of Concerns**
- Frontend handles presentation and user interaction only
- Backend manages business logic, validation, and orchestration
- ML service focuses exclusively on data science operations
- Database layer provides persistent storage abstraction

**Stateless Services**
- All application servers are stateless for horizontal scaling
- Session state stored in Redis or JWT tokens
- No server-side session affinity required

**API-First Design**
- All functionality exposed via RESTful APIs
- Versioned endpoints for backward compatibility
- Comprehensive OpenAPI documentation

**Microservices-Oriented**
- ML service independently deployable and scalable
- Clear service boundaries with well-defined contracts
- Inter-service communication via HTTP/REST

**Cloud-Native**
- Containerized deployment (Docker)
- Infrastructure as Code (Terraform/CloudFormation)
- Managed services for databases and caching
- Auto-scaling based on metrics


## 3. Component-Level Design

### 3.1 Frontend Architecture (React SPA)

#### 3.1.1 Technology Stack
```javascript
{
  "core": "React 18.2",
  "buildTool": "Vite 4.x",
  "routing": "react-router-dom 6.x",
  "stateManagement": "React Context + React Query",
  "uiFramework": "Material-UI (MUI) 5.x",
  "charts": "Recharts 2.x",
  "forms": "React Hook Form + Zod",
  "http": "Axios",
  "dateHandling": "date-fns"
}
```

#### 3.1.2 Directory Structure
```
client/
├── src/
│   ├── components/          # Reusable UI components
│   │   ├── common/          # Buttons, Inputs, Cards
│   │   ├── charts/          # Chart components
│   │   └── layout/          # Header, Sidebar, Footer
│   ├── pages/               # Route-level components
│   │   ├── Dashboard/
│   │   ├── Products/
│   │   ├── Sales/
│   │   ├── Analytics/
│   │   └── Auth/
│   ├── hooks/               # Custom React hooks
│   │   ├── useAuth.js
│   │   ├── useProducts.js
│   │   └── useAnalytics.js
│   ├── services/            # API client services
│   │   ├── api.js           # Axios instance
│   │   ├── authService.js
│   │   ├── productService.js
│   │   └── analyticsService.js
│   ├── context/             # React Context providers
│   │   ├── AuthContext.jsx
│   │   └── ThemeContext.jsx
│   ├── utils/               # Utility functions
│   │   ├── formatters.js
│   │   ├── validators.js
│   │   └── constants.js
│   ├── routes/              # Route configuration
│   │   └── index.jsx
│   ├── App.jsx              # Root component
│   └── main.jsx             # Entry point
├── public/
├── package.json
└── vite.config.js
```

#### 3.1.3 Key Components

**Authentication Flow**
```javascript
// AuthContext.jsx - Manages authentication state
const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Check for existing token on mount
    const token = localStorage.getItem('token');
    if (token) {
      validateToken(token);
    }
    setLoading(false);
  }, []);

  const login = async (email, password) => {
    const response = await authService.login(email, password);
    localStorage.setItem('token', response.token);
    setUser(response.user);
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, loading }}>
      {children}
    </AuthContext.Provider>
  );
};
```

**API Service Layer**
```javascript
// services/api.js - Axios configuration
import axios from 'axios';

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10000,
});

// Request interceptor - Add auth token
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response interceptor - Handle errors
api.interceptors.response.use(
  (response) => response.data,
  (error) => {
    if (error.response?.status === 401) {
      // Redirect to login
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default api;
```

**Data Fetching with React Query**
```javascript
// hooks/useProducts.js
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import productService from '../services/productService';

export const useProducts = () => {
  return useQuery({
    queryKey: ['products'],
    queryFn: productService.getAll,
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
};

export const useCreateProduct = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: productService.create,
    onSuccess: () => {
      queryClient.invalidateQueries(['products']);
    },
  });
};
```

#### 3.1.4 State Management Strategy

**Local Component State**: useState for UI-only state (modals, form inputs)  
**Global App State**: React Context for auth, theme, user preferences  
**Server State**: React Query for API data with caching and synchronization  
**Form State**: React Hook Form for complex forms with validation


### 3.2 Backend API Architecture (Node.js/Express)

#### 3.2.1 Technology Stack
```javascript
{
  "runtime": "Node.js 18 LTS",
  "framework": "Express.js 4.18",
  "database": "MongoDB 5.0 + Mongoose 8.0",
  "authentication": "jsonwebtoken + bcryptjs",
  "validation": "express-validator",
  "logging": "winston",
  "testing": "jest + supertest"
}
```

#### 3.2.2 Directory Structure
```
server/
├── src/
│   ├── config/              # Configuration files
│   │   ├── db.js            # Database connection
│   │   ├── redis.js         # Redis client
│   │   └── constants.js     # App constants
│   ├── models/              # Mongoose models
│   │   ├── User.js
│   │   ├── Product.js
│   │   ├── Sale.js
│   │   ├── Customer.js
│   │   └── Forecast.js
│   ├── controllers/         # Request handlers
│   │   ├── authController.js
│   │   ├── productController.js
│   │   ├── saleController.js
│   │   └── analyticsController.js
│   ├── services/            # Business logic layer
│   │   ├── authService.js
│   │   ├── productService.js
│   │   ├── saleService.js
│   │   ├── analyticsService.js
│   │   └── mlService.js     # ML service client
│   ├── middleware/          # Express middleware
│   │   ├── auth.js          # JWT verification
│   │   ├── errorHandler.js  # Global error handler
│   │   ├── validator.js     # Request validation
│   │   └── rateLimiter.js   # Rate limiting
│   ├── routes/              # Route definitions
│   │   ├── auth.js
│   │   ├── products.js
│   │   ├── sales.js
│   │   └── analytics.js
│   ├── utils/               # Utility functions
│   │   ├── logger.js
│   │   ├── asyncHandler.js
│   │   └── apiResponse.js
│   ├── jobs/                # Background jobs
│   │   ├── dailyForecast.js
│   │   └── reportGenerator.js
│   └── index.js             # Application entry point
├── tests/
│   ├── unit/
│   └── integration/
├── package.json
└── .env.example
```

#### 3.2.3 Layered Architecture

**Layer 1: Routes** - Define endpoints and map to controllers
```javascript
// routes/products.js
import express from 'express';
import { authenticate } from '../middleware/auth.js';
import { validateProduct } from '../middleware/validator.js';
import * as productController from '../controllers/productController.js';

const router = express.Router();

router.use(authenticate); // All routes require authentication

router.get('/', productController.getAll);
router.get('/:id', productController.getById);
router.post('/', validateProduct, productController.create);
router.put('/:id', validateProduct, productController.update);
router.delete('/:id', productController.remove);
router.post('/bulk-import', productController.bulkImport);

export default router;
```

**Layer 2: Controllers** - Handle HTTP requests/responses
```javascript
// controllers/productController.js
import * as productService from '../services/productService.js';
import { asyncHandler } from '../utils/asyncHandler.js';
import { ApiResponse } from '../utils/apiResponse.js';

export const getAll = asyncHandler(async (req, res) => {
  const { page = 1, limit = 50, category, search } = req.query;
  
  const result = await productService.getProducts(req.userId, {
    page: parseInt(page),
    limit: parseInt(limit),
    category,
    search,
  });

  res.json(ApiResponse.success(result));
});

export const create = asyncHandler(async (req, res) => {
  const product = await productService.createProduct(req.userId, req.body);
  res.status(201).json(ApiResponse.success(product, 'Product created'));
});
```

**Layer 3: Services** - Business logic and data operations
```javascript
// services/productService.js
import Product from '../models/Product.js';
import { AppError } from '../utils/appError.js';

export const getProducts = async (userId, options) => {
  const { page, limit, category, search } = options;
  
  const query = { userId };
  if (category) query.category = category;
  if (search) {
    query.$or = [
      { name: { $regex: search, $options: 'i' } },
      { sku: { $regex: search, $options: 'i' } },
    ];
  }

  const skip = (page - 1) * limit;
  
  const [products, total] = await Promise.all([
    Product.find(query).skip(skip).limit(limit).sort({ createdAt: -1 }),
    Product.countDocuments(query),
  ]);

  return {
    products,
    pagination: {
      page,
      limit,
      total,
      pages: Math.ceil(total / limit),
    },
  };
};

export const createProduct = async (userId, data) => {
  // Check SKU uniqueness
  const existing = await Product.findOne({ userId, sku: data.sku });
  if (existing) {
    throw new AppError('SKU already exists', 400);
  }

  const product = await Product.create({ ...data, userId });
  return product;
};
```

**Layer 4: Models** - Data schema and validation
```javascript
// models/Product.js
import mongoose from 'mongoose';

const productSchema = new mongoose.Schema({
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true,
    index: true,
  },
  name: {
    type: String,
    required: [true, 'Product name is required'],
    trim: true,
    maxlength: 200,
  },
  sku: {
    type: String,
    required: [true, 'SKU is required'],
    trim: true,
    uppercase: true,
  },
  category: {
    type: String,
    trim: true,
    index: true,
  },
  currentPrice: {
    type: Number,
    required: [true, 'Price is required'],
    min: 0,
  },
  costPrice: {
    type: Number,
    min: 0,
  },
  stock: {
    type: Number,
    default: 0,
    min: 0,
  },
  reorderLevel: {
    type: Number,
    default: 10,
    min: 0,
  },
  priceHistory: [{
    price: Number,
    date: { type: Date, default: Date.now },
  }],
  isActive: {
    type: Boolean,
    default: true,
  },
}, {
  timestamps: true,
});

// Compound index for userId + sku uniqueness
productSchema.index({ userId: 1, sku: 1 }, { unique: true });

// Virtual for profit margin
productSchema.virtual('profitMargin').get(function() {
  if (!this.costPrice) return null;
  return ((this.currentPrice - this.costPrice) / this.currentPrice) * 100;
});

export default mongoose.model('Product', productSchema);
```

#### 3.2.4 Error Handling Strategy

**Custom Error Class**
```javascript
// utils/appError.js
export class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}
```

**Global Error Handler Middleware**
```javascript
// middleware/errorHandler.js
import { logger } from '../utils/logger.js';

export const errorHandler = (err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || 'error';

  // Log error
  logger.error({
    message: err.message,
    stack: err.stack,
    url: req.originalUrl,
    method: req.method,
    userId: req.userId,
  });

  // Development vs Production error response
  if (process.env.NODE_ENV === 'development') {
    res.status(err.statusCode).json({
      status: err.status,
      error: err,
      message: err.message,
      stack: err.stack,
    });
  } else {
    // Production - don't leak error details
    if (err.isOperational) {
      res.status(err.statusCode).json({
        status: err.status,
        message: err.message,
      });
    } else {
      // Programming or unknown error
      res.status(500).json({
        status: 'error',
        message: 'Something went wrong',
      });
    }
  }
};
```

#### 3.2.5 ML Service Integration

```javascript
// services/mlService.js
import axios from 'axios';
import { logger } from '../utils/logger.js';

const mlClient = axios.create({
  baseURL: process.env.ML_SERVICE_URL,
  timeout: 30000, // 30 seconds for ML operations
});

export const getForecast = async (productId, salesData) => {
  try {
    const response = await mlClient.post('/predict/demand', {
      product_id: productId,
      sales_history: salesData,
      forecast_days: 30,
    });
    return response.data;
  } catch (error) {
    logger.error('ML service error:', error);
    throw new AppError('Forecast service unavailable', 503);
  }
};

export const getPriceRecommendation = async (productData) => {
  try {
    const response = await mlClient.post('/predict/price', productData);
    return response.data;
  } catch (error) {
    logger.error('ML service error:', error);
    // Return fallback recommendation
    return {
      recommended_price: productData.current_price,
      confidence: 0,
      fallback: true,
    };
  }
};
```


### 3.3 ML Service Architecture (Python/Flask)

#### 3.3.1 Technology Stack
```python
{
  "framework": "Flask 2.3",
  "wsgi": "Gunicorn",
  "ml_libraries": [
    "scikit-learn 1.3",
    "pandas 2.0",
    "numpy 1.24",
    "statsmodels 0.14",
    "prophet 1.1"
  ],
  "model_storage": "joblib",
  "api_docs": "Flask-RESTX",
  "testing": "pytest"
}
```

#### 3.3.2 Directory Structure
```
ml-service/
├── app/
│   ├── __init__.py          # Flask app factory
│   ├── config.py            # Configuration
│   ├── models/              # ML models
│   │   ├── __init__.py
│   │   ├── demand_forecaster.py
│   │   ├── price_optimizer.py
│   │   └── customer_segmenter.py
│   ├── services/            # Business logic
│   │   ├── __init__.py
│   │   ├── forecast_service.py
│   │   ├── pricing_service.py
│   │   └── segmentation_service.py
│   ├── utils/               # Utilities
│   │   ├── __init__.py
│   │   ├── data_processor.py
│   │   ├── validators.py
│   │   └── model_loader.py
│   ├── routes/              # API endpoints
│   │   ├── __init__.py
│   │   ├── predict.py
│   │   └── train.py
│   └── schemas/             # Request/response schemas
│       ├── __init__.py
│       └── prediction_schemas.py
├── models/                  # Saved model files
│   ├── demand_forecaster_v1.pkl
│   └── metadata.json
├── data/                    # Training data cache
├── tests/
│   ├── unit/
│   └── integration/
├── requirements.txt
├── app.py                   # Entry point
└── gunicorn.conf.py
```

#### 3.3.3 Flask Application Structure

**Application Factory Pattern**
```python
# app/__init__.py
from flask import Flask
from flask_cors import CORS
from app.routes import predict_bp, train_bp
from app.config import Config

def create_app(config_class=Config):
    app = Flask(__name__)
    app.config.from_object(config_class)
    
    # Enable CORS
    CORS(app, origins=app.config['ALLOWED_ORIGINS'])
    
    # Register blueprints
    app.register_blueprint(predict_bp, url_prefix='/predict')
    app.register_blueprint(train_bp, url_prefix='/train')
    
    # Health check endpoint
    @app.route('/health')
    def health():
        return {'status': 'healthy', 'service': 'ml-service'}
    
    return app
```

**Demand Forecasting Model**
```python
# app/models/demand_forecaster.py
import pandas as pd
import numpy as np
from statsmodels.tsa.statespace.sarimax import SARIMAX
from prophet import Prophet
import joblib
from datetime import datetime, timedelta

class DemandForecaster:
    """
    Ensemble demand forecasting using SARIMA and Prophet
    """
    
    def __init__(self):
        self.sarima_model = None
        self.prophet_model = None
        self.model_weights = {'sarima': 0.5, 'prophet': 0.5}
    
    def prepare_data(self, sales_history):
        """
        Convert sales history to time series format
        
        Args:
            sales_history: List of dicts with 'date' and 'quantity'
        
        Returns:
            pandas DataFrame with datetime index
        """
        df = pd.DataFrame(sales_history)
        df['date'] = pd.to_datetime(df['date'])
        df = df.set_index('date')
        df = df.resample('D').sum().fillna(0)  # Daily aggregation
        return df
    
    def detect_seasonality(self, data, threshold=0.6):
        """
        Detect if data has seasonal patterns
        """
        from scipy import stats
        
        if len(data) < 60:  # Need at least 2 months
            return False
        
        # Simple autocorrelation check
        weekly_corr = data['quantity'].autocorr(lag=7)
        return abs(weekly_corr) > threshold
    
    def train_sarima(self, data):
        """
        Train SARIMA model for time series with seasonality
        """
        try:
            # Auto-detect best parameters (simplified)
            order = (1, 1, 1)
            seasonal_order = (1, 1, 1, 7)  # Weekly seasonality
            
            self.sarima_model = SARIMAX(
                data['quantity'],
                order=order,
                seasonal_order=seasonal_order,
                enforce_stationarity=False,
                enforce_invertibility=False
            )
            self.sarima_model = self.sarima_model.fit(disp=False)
            return True
        except Exception as e:
            print(f"SARIMA training failed: {e}")
            return False
    
    def train_prophet(self, data):
        """
        Train Prophet model (robust to missing data and outliers)
        """
        try:
            # Prophet requires specific column names
            prophet_df = data.reset_index()
            prophet_df.columns = ['ds', 'y']
            
            self.prophet_model = Prophet(
                daily_seasonality=False,
                weekly_seasonality=True,
                yearly_seasonality=True if len(data) > 365 else False,
                changepoint_prior_scale=0.05
            )
            self.prophet_model.fit(prophet_df)
            return True
        except Exception as e:
            print(f"Prophet training failed: {e}")
            return False
    
    def predict(self, sales_history, forecast_days=30):
        """
        Generate demand forecast
        
        Args:
            sales_history: List of sales records
            forecast_days: Number of days to forecast
        
        Returns:
            dict with predictions and confidence intervals
        """
        # Prepare data
        data = self.prepare_data(sales_history)
        
        if len(data) < 30:
            return {
                'error': 'Insufficient data',
                'message': 'Need at least 30 days of sales history',
                'forecast': []
            }
        
        # Train models
        has_seasonality = self.detect_seasonality(data)
        
        predictions = []
        
        # SARIMA prediction
        if has_seasonality and self.train_sarima(data):
            sarima_pred = self.sarima_model.forecast(steps=forecast_days)
            predictions.append(('sarima', sarima_pred))
        
        # Prophet prediction (always try)
        if self.train_prophet(data):
            future = self.prophet_model.make_future_dataframe(periods=forecast_days)
            prophet_pred = self.prophet_model.predict(future)
            prophet_pred = prophet_pred.tail(forecast_days)['yhat'].values
            predictions.append(('prophet', prophet_pred))
        
        if not predictions:
            # Fallback: simple moving average
            ma = data['quantity'].rolling(window=7).mean().iloc[-1]
            forecast = [max(0, ma) for _ in range(forecast_days)]
            confidence = 0.3
        else:
            # Ensemble: weighted average
            forecast = np.zeros(forecast_days)
            total_weight = 0
            for model_name, pred in predictions:
                weight = self.model_weights.get(model_name, 0.5)
                forecast += weight * pred
                total_weight += weight
            forecast = forecast / total_weight
            forecast = np.maximum(forecast, 0)  # No negative demand
            confidence = 0.7 if len(predictions) > 1 else 0.5
        
        # Generate date range
        last_date = data.index[-1]
        future_dates = [
            (last_date + timedelta(days=i+1)).strftime('%Y-%m-%d')
            for i in range(forecast_days)
        ]
        
        # Calculate confidence intervals (simplified)
        std = data['quantity'].std()
        lower_bound = forecast - 1.96 * std
        upper_bound = forecast + 1.96 * std
        
        return {
            'forecast': [
                {
                    'date': date,
                    'predicted_demand': float(pred),
                    'lower_bound': max(0, float(lower)),
                    'upper_bound': float(upper)
                }
                for date, pred, lower, upper in zip(
                    future_dates, forecast, lower_bound, upper_bound
                )
            ],
            'confidence': confidence,
            'model_used': 'ensemble' if len(predictions) > 1 else predictions[0][0],
            'has_seasonality': has_seasonality
        }
    
    def save(self, filepath):
        """Save trained models"""
        joblib.dump({
            'sarima': self.sarima_model,
            'prophet': self.prophet_model,
            'weights': self.model_weights
        }, filepath)
    
    def load(self, filepath):
        """Load trained models"""
        data = joblib.load(filepath)
        self.sarima_model = data['sarima']
        self.prophet_model = data['prophet']
        self.model_weights = data['weights']
```


**Price Optimization Model**
```python
# app/models/price_optimizer.py
import numpy as np
from scipy.optimize import minimize
from sklearn.linear_model import LinearRegression

class PriceOptimizer:
    """
    Dynamic pricing optimization using price elasticity
    """
    
    def __init__(self):
        self.elasticity_model = None
        self.price_elasticity = None
    
    def estimate_price_elasticity(self, price_history, demand_history):
        """
        Estimate price elasticity of demand
        
        Price Elasticity = % change in demand / % change in price
        """
        if len(price_history) < 10:
            return None  # Insufficient data
        
        # Calculate percentage changes
        prices = np.array(price_history)
        demands = np.array(demand_history)
        
        # Log-log regression for elasticity
        log_prices = np.log(prices + 1).reshape(-1, 1)
        log_demands = np.log(demands + 1)
        
        self.elasticity_model = LinearRegression()
        self.elasticity_model.fit(log_prices, log_demands)
        
        # Coefficient is the elasticity
        self.price_elasticity = self.elasticity_model.coef_[0]
        
        return self.price_elasticity
    
    def predict_demand_at_price(self, current_price, current_demand, new_price):
        """
        Predict demand at a different price point
        """
        if self.price_elasticity is None:
            return current_demand  # No elasticity data
        
        price_change_pct = (new_price - current_price) / current_price
        demand_change_pct = self.price_elasticity * price_change_pct
        
        predicted_demand = current_demand * (1 + demand_change_pct)
        return max(0, predicted_demand)
    
    def optimize_price(self, product_data, strategy='profit'):
        """
        Find optimal price based on strategy
        
        Args:
            product_data: dict with current_price, cost_price, current_demand,
                         price_history, demand_history, min_price, max_price
            strategy: 'profit' | 'revenue' | 'volume' | 'clearance'
        
        Returns:
            dict with recommended_price, expected_demand, expected_profit
        """
        current_price = product_data['current_price']
        cost_price = product_data.get('cost_price', current_price * 0.6)
        current_demand = product_data['current_demand']
        min_price = product_data.get('min_price', cost_price * 1.1)
        max_price = product_data.get('max_price', current_price * 2)
        stock = product_data.get('stock', 100)
        
        # Estimate elasticity if data available
        if 'price_history' in product_data and 'demand_history' in product_data:
            self.estimate_price_elasticity(
                product_data['price_history'],
                product_data['demand_history']
            )
        
        # Define objective function based on strategy
        def objective(price):
            demand = self.predict_demand_at_price(
                current_price, current_demand, price[0]
            )
            
            if strategy == 'profit':
                return -(price[0] - cost_price) * demand  # Maximize profit
            elif strategy == 'revenue':
                return -price[0] * demand  # Maximize revenue
            elif strategy == 'volume':
                return -demand  # Maximize volume
            elif strategy == 'clearance':
                # Maximize revenue while clearing stock quickly
                days_to_clear = stock / (demand + 1)
                penalty = max(0, days_to_clear - 30) * 10  # Penalty if >30 days
                return -(price[0] * demand) + penalty
            else:
                return -(price[0] - cost_price) * demand
        
        # Optimize
        result = minimize(
            objective,
            x0=[current_price],
            bounds=[(min_price, max_price)],
            method='L-BFGS-B'
        )
        
        optimal_price = result.x[0]
        expected_demand = self.predict_demand_at_price(
            current_price, current_demand, optimal_price
        )
        expected_profit = (optimal_price - cost_price) * expected_demand
        expected_revenue = optimal_price * expected_demand
        
        # Calculate confidence based on data availability
        confidence = 0.8 if self.price_elasticity is not None else 0.4
        
        return {
            'recommended_price': round(optimal_price, 2),
            'current_price': current_price,
            'price_change_pct': round(
                ((optimal_price - current_price) / current_price) * 100, 2
            ),
            'expected_demand': round(expected_demand, 1),
            'expected_profit': round(expected_profit, 2),
            'expected_revenue': round(expected_revenue, 2),
            'confidence': confidence,
            'strategy': strategy,
            'price_elasticity': self.price_elasticity
        }
```

**Customer Segmentation Model**
```python
# app/models/customer_segmenter.py
import pandas as pd
import numpy as np
from sklearn.preprocessing import StandardScaler
from sklearn.cluster import KMeans
from datetime import datetime, timedelta

class CustomerSegmenter:
    """
    RFM (Recency, Frequency, Monetary) customer segmentation
    """
    
    def __init__(self, n_segments=4):
        self.n_segments = n_segments
        self.scaler = StandardScaler()
        self.kmeans = None
        self.segment_labels = {
            0: 'Champions',
            1: 'Loyal Customers',
            2: 'At Risk',
            3: 'Lost'
        }
    
    def calculate_rfm(self, customer_transactions):
        """
        Calculate RFM metrics for each customer
        
        Args:
            customer_transactions: List of dicts with customer_id, date, amount
        
        Returns:
            DataFrame with RFM scores
        """
        df = pd.DataFrame(customer_transactions)
        df['date'] = pd.to_datetime(df['date'])
        
        # Reference date (today)
        reference_date = df['date'].max() + timedelta(days=1)
        
        # Calculate RFM
        rfm = df.groupby('customer_id').agg({
            'date': lambda x: (reference_date - x.max()).days,  # Recency
            'customer_id': 'count',  # Frequency
            'amount': 'sum'  # Monetary
        })
        
        rfm.columns = ['recency', 'frequency', 'monetary']
        
        # Calculate RFM scores (1-5 scale)
        rfm['r_score'] = pd.qcut(rfm['recency'], 5, labels=[5,4,3,2,1], duplicates='drop')
        rfm['f_score'] = pd.qcut(rfm['frequency'], 5, labels=[1,2,3,4,5], duplicates='drop')
        rfm['m_score'] = pd.qcut(rfm['monetary'], 5, labels=[1,2,3,4,5], duplicates='drop')
        
        # Overall RFM score
        rfm['rfm_score'] = (
            rfm['r_score'].astype(int) +
            rfm['f_score'].astype(int) +
            rfm['m_score'].astype(int)
        )
        
        return rfm
    
    def segment_customers(self, rfm_data):
        """
        Segment customers using K-means clustering
        """
        # Prepare features for clustering
        features = rfm_data[['recency', 'frequency', 'monetary']].values
        features_scaled = self.scaler.fit_transform(features)
        
        # K-means clustering
        self.kmeans = KMeans(n_clusters=self.n_segments, random_state=42)
        rfm_data['segment'] = self.kmeans.fit_predict(features_scaled)
        
        # Assign segment labels based on characteristics
        segment_summary = rfm_data.groupby('segment').agg({
            'recency': 'mean',
            'frequency': 'mean',
            'monetary': 'mean'
        })
        
        # Sort by value (low recency, high frequency, high monetary = best)
        segment_summary['value_score'] = (
            -segment_summary['recency'] +
            segment_summary['frequency'] +
            segment_summary['monetary']
        )
        segment_summary = segment_summary.sort_values('value_score', ascending=False)
        
        # Map segments to labels
        segment_mapping = {
            old: new for new, old in enumerate(segment_summary.index)
        }
        rfm_data['segment'] = rfm_data['segment'].map(segment_mapping)
        rfm_data['segment_label'] = rfm_data['segment'].map(self.segment_labels)
        
        return rfm_data
    
    def calculate_clv(self, rfm_data, time_horizon_months=12):
        """
        Calculate Customer Lifetime Value (simplified)
        
        CLV = (Average Purchase Value × Purchase Frequency) × Customer Lifespan
        """
        avg_purchase_value = rfm_data['monetary'] / rfm_data['frequency']
        purchase_frequency_monthly = rfm_data['frequency'] / (rfm_data['recency'] / 30)
        
        # Estimate lifespan based on recency (customers with low recency likely to stay)
        estimated_lifespan_months = np.where(
            rfm_data['recency'] < 30, time_horizon_months,
            np.where(rfm_data['recency'] < 90, time_horizon_months * 0.7,
                    time_horizon_months * 0.3)
        )
        
        clv = avg_purchase_value * purchase_frequency_monthly * estimated_lifespan_months
        rfm_data['clv'] = clv
        
        return rfm_data
    
    def analyze(self, customer_transactions):
        """
        Complete customer segmentation analysis
        """
        # Calculate RFM
        rfm = self.calculate_rfm(customer_transactions)
        
        # Segment customers
        rfm = self.segment_customers(rfm)
        
        # Calculate CLV
        rfm = self.calculate_clv(rfm)
        
        # Generate insights
        segment_stats = rfm.groupby('segment_label').agg({
            'customer_id': 'count',
            'monetary': 'sum',
            'clv': 'sum',
            'recency': 'mean',
            'frequency': 'mean'
        }).round(2)
        
        segment_stats.columns = [
            'customer_count', 'total_revenue', 'total_clv',
            'avg_recency', 'avg_frequency'
        ]
        
        return {
            'customer_segments': rfm.reset_index().to_dict('records'),
            'segment_summary': segment_stats.to_dict('index'),
            'total_customers': len(rfm),
            'total_clv': float(rfm['clv'].sum())
        }
```


**API Routes**
```python
# app/routes/predict.py
from flask import Blueprint, request, jsonify
from app.models.demand_forecaster import DemandForecaster
from app.models.price_optimizer import PriceOptimizer
from app.models.customer_segmenter import CustomerSegmenter
from app.utils.validators import validate_forecast_request

predict_bp = Blueprint('predict', __name__)

@predict_bp.route('/demand', methods=['POST'])
def predict_demand():
    """
    Predict product demand
    
    Request body:
    {
        "product_id": "string",
        "sales_history": [{"date": "2024-01-01", "quantity": 10}, ...],
        "forecast_days": 30
    }
    """
    try:
        data = request.get_json()
        
        # Validate request
        errors = validate_forecast_request(data)
        if errors:
            return jsonify({'error': 'Validation failed', 'details': errors}), 400
        
        # Generate forecast
        forecaster = DemandForecaster()
        result = forecaster.predict(
            sales_history=data['sales_history'],
            forecast_days=data.get('forecast_days', 30)
        )
        
        return jsonify(result), 200
        
    except Exception as e:
        return jsonify({'error': str(e)}), 500


@predict_bp.route('/price', methods=['POST'])
def predict_price():
    """
    Recommend optimal price
    
    Request body:
    {
        "product_id": "string",
        "current_price": 100,
        "cost_price": 60,
        "current_demand": 50,
        "price_history": [95, 100, 105],
        "demand_history": [55, 50, 45],
        "strategy": "profit",
        "min_price": 70,
        "max_price": 150,
        "stock": 200
    }
    """
    try:
        data = request.get_json()
        
        optimizer = PriceOptimizer()
        result = optimizer.optimize_price(
            product_data=data,
            strategy=data.get('strategy', 'profit')
        )
        
        return jsonify(result), 200
        
    except Exception as e:
        return jsonify({'error': str(e)}), 500


@predict_bp.route('/customer-segments', methods=['POST'])
def segment_customers():
    """
    Segment customers using RFM analysis
    
    Request body:
    {
        "transactions": [
            {"customer_id": "C001", "date": "2024-01-01", "amount": 100},
            ...
        ]
    }
    """
    try:
        data = request.get_json()
        
        segmenter = CustomerSegmenter()
        result = segmenter.analyze(data['transactions'])
        
        return jsonify(result), 200
        
    except Exception as e:
        return jsonify({'error': str(e)}), 500
```


## 4. Data Flow Architecture

### 4.1 Request Flow Diagram

```
┌──────────────┐
│   Browser    │
│  (React SPA) │
└──────┬───────┘
       │ 1. User Action (e.g., "Get Forecast")
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│  Frontend State Management                                │
│  - Validate input                                         │
│  - Show loading state                                     │
│  - Prepare API request                                    │
└──────┬───────────────────────────────────────────────────┘
       │ 2. HTTP POST /api/analytics/forecast/:productId
       │    Headers: { Authorization: "Bearer <JWT>" }
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│  API Gateway / Load Balancer                              │
│  - SSL termination                                        │
│  - Rate limiting check                                    │
│  - Route to backend instance                              │
└──────┬───────────────────────────────────────────────────┘
       │ 3. Forward request
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│  Express Middleware Chain                                 │
│  ┌────────────────────────────────────────────────────┐  │
│  │ 1. CORS Middleware                                 │  │
│  │    - Validate origin                               │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────┐  │
│  │ 2. Auth Middleware                                 │  │
│  │    - Verify JWT token                              │  │
│  │    - Extract userId                                │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────┐  │
│  │ 3. Request Validation                              │  │
│  │    - Validate productId format                     │  │
│  └────────────────────────────────────────────────────┘  │
└──────┬───────────────────────────────────────────────────┘
       │ 4. Request reaches controller
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│  Analytics Controller                                     │
│  - Extract parameters                                     │
│  - Call analytics service                                 │
└──────┬───────────────────────────────────────────────────┘
       │ 5. Service layer call
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│  Analytics Service                                        │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Step 1: Fetch product data                         │  │
│  │         Query MongoDB for product details          │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Step 2: Fetch sales history                        │  │
│  │         Query MongoDB for last 90 days sales       │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Step 3: Check cache                                │  │
│  │         Redis: GET forecast:{productId}:{date}     │  │
│  │         If found, return cached result             │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Step 4: Call ML service                            │  │
│  │         POST to ML service /predict/demand         │  │
│  └────────────────────────────────────────────────────┘  │
└──────┬───────────────────────────────────────────────────┘
       │ 6. HTTP request to ML service
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│  ML Service (Flask)                                       │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Step 1: Validate input data                        │  │
│  │         - Check data format                        │  │
│  │         - Verify minimum data requirements         │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Step 2: Data preprocessing                         │  │
│  │         - Convert to time series                   │  │
│  │         - Handle missing values                    │  │
│  │         - Detect seasonality                       │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Step 3: Model selection                            │  │
│  │         - Choose SARIMA/Prophet based on data      │  │
│  │         - Load pre-trained models if available     │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Step 4: Generate predictions                       │  │
│  │         - Run forecasting models                   │  │
│  │         - Calculate confidence intervals           │  │
│  │         - Ensemble results                         │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Step 5: Format response                            │  │
│  │         - Structure prediction data                │  │
│  │         - Add metadata                             │  │
│  └────────────────────────────────────────────────────┘  │
└──────┬───────────────────────────────────────────────────┘
       │ 7. Return predictions
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│  Analytics Service (continued)                            │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Step 5: Process ML response                        │  │
│  │         - Validate predictions                     │  │
│  │         - Enrich with business context             │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Step 6: Cache results                              │  │
│  │         Redis: SET forecast:{productId}:{date}     │  │
│  │         TTL: 24 hours                              │  │
│  └────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Step 7: Save to database                           │  │
│  │         MongoDB: Insert forecast record            │  │
│  └────────────────────────────────────────────────────┘  │
└──────┬───────────────────────────────────────────────────┘
       │ 8. Return to controller
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│  Analytics Controller                                     │
│  - Format API response                                    │
│  - Add response headers                                   │
└──────┬───────────────────────────────────────────────────┘
       │ 9. HTTP 200 OK with JSON response
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│  Frontend (React)                                         │
│  - Update React Query cache                               │
│  - Update component state                                 │
│  - Render forecast chart                                  │
│  - Hide loading state                                     │
└───────────────────────────────────────────────────────────┘
```

### 4.2 Data Synchronization Flow

**Sales Transaction Recording**
```
User enters sale → Frontend validates → API creates Sale record →
→ Update Product.stock (decrement) → Invalidate forecast cache →
→ Trigger background job to retrain model (if needed)
```

**Daily Forecast Update (Background Job)**
```
Cron job triggers (2 AM daily) → Fetch all active products →
→ For each product: fetch recent sales → Call ML service →
→ Save forecast to DB → Update cache → Send alerts if needed
```

### 4.3 Caching Strategy

**Cache Layers:**

1. **Browser Cache**: Static assets (JS, CSS, images) - 1 year
2. **CDN Cache**: Frontend build artifacts - 1 month
3. **Redis Cache**: 
   - API responses (forecasts, analytics) - 24 hours
   - Session data - 24 hours
   - Rate limit counters - 1 minute
4. **React Query Cache**: API data - 5 minutes (stale time)

**Cache Invalidation:**
- Product update → Invalidate product cache + forecast cache
- New sale → Invalidate product cache + analytics cache
- Price change → Invalidate pricing recommendations


## 5. Database Design

### 5.1 MongoDB Collections Schema

#### 5.1.1 Users Collection

```javascript
{
  _id: ObjectId("..."),
  email: "user@example.com",
  password: "$2a$10$...",  // bcrypt hash
  businessName: "ABC Retail Store",
  role: "owner",  // enum: ['owner', 'manager']
  profile: {
    firstName: "John",
    lastName: "Doe",
    phone: "+1234567890",
    timezone: "America/New_York"
  },
  settings: {
    currency: "USD",
    dateFormat: "MM/DD/YYYY",
    notifications: {
      email: true,
      lowStock: true,
      priceAlerts: true
    }
  },
  subscription: {
    plan: "professional",  // enum: ['starter', 'professional', 'enterprise']
    status: "active",
    startDate: ISODate("2024-01-01"),
    renewalDate: ISODate("2024-02-01")
  },
  createdAt: ISODate("2024-01-01T00:00:00Z"),
  updatedAt: ISODate("2024-01-15T10:30:00Z"),
  lastLoginAt: ISODate("2024-01-15T10:30:00Z")
}

// Indexes
db.users.createIndex({ email: 1 }, { unique: true })
db.users.createIndex({ createdAt: -1 })
```

#### 5.1.2 Products Collection

```javascript
{
  _id: ObjectId("..."),
  userId: ObjectId("..."),  // Reference to users
  name: "Wireless Mouse",
  sku: "WM-001",
  description: "Ergonomic wireless mouse with USB receiver",
  category: "Electronics",
  subcategory: "Computer Accessories",
  
  // Pricing
  currentPrice: 29.99,
  costPrice: 15.00,
  currency: "USD",
  priceHistory: [
    { price: 34.99, date: ISODate("2024-01-01"), reason: "Initial price" },
    { price: 29.99, date: ISODate("2024-01-15"), reason: "Promotional discount" }
  ],
  
  // Inventory
  stock: 150,
  reorderLevel: 20,
  reorderQuantity: 100,
  unit: "piece",
  
  // Supplier info
  supplier: {
    name: "Tech Supplies Inc",
    sku: "TS-WM-001",
    leadTimeDays: 7
  },
  
  // Product attributes
  attributes: {
    brand: "TechBrand",
    color: "Black",
    weight: "0.2 kg",
    dimensions: "10x6x4 cm"
  },
  
  // Images
  images: [
    "https://cdn.example.com/products/wm001-1.jpg",
    "https://cdn.example.com/products/wm001-2.jpg"
  ],
  
  // Metadata
  isActive: true,
  tags: ["wireless", "ergonomic", "office"],
  
  // Analytics cache (updated daily)
  analytics: {
    totalSales: 450,
    totalRevenue: 13497.00,
    avgDailySales: 5.2,
    lastSaleDate: ISODate("2024-01-14"),
    inventoryTurnover: 3.0,
    daysOfInventory: 28,
    updatedAt: ISODate("2024-01-15T02:00:00Z")
  },
  
  createdAt: ISODate("2024-01-01T00:00:00Z"),
  updatedAt: ISODate("2024-01-15T08:00:00Z")
}

// Indexes
db.products.createIndex({ userId: 1, sku: 1 }, { unique: true })
db.products.createIndex({ userId: 1, category: 1 })
db.products.createIndex({ userId: 1, isActive: 1 })
db.products.createIndex({ userId: 1, stock: 1 })  // For low stock queries
db.products.createIndex({ name: "text", description: "text" })  // Text search
```

#### 5.1.3 Sales Collection

```javascript
{
  _id: ObjectId("..."),
  userId: ObjectId("..."),
  productId: ObjectId("..."),
  
  // Transaction details
  quantity: 2,
  unitPrice: 29.99,
  totalAmount: 59.98,
  discount: 0,
  tax: 4.80,
  finalAmount: 64.78,
  
  // Customer info
  customerId: "CUST-001",  // Optional, for customer tracking
  customerName: "Jane Smith",
  
  // Sale metadata
  saleDate: ISODate("2024-01-15T14:30:00Z"),
  channel: "in-store",  // enum: ['in-store', 'online', 'phone']
  paymentMethod: "credit_card",
  transactionId: "TXN-20240115-001",
  
  // Location (for multi-store)
  storeLocation: "Main Street Store",
  
  // Notes
  notes: "Customer requested gift wrapping",
  
  createdAt: ISODate("2024-01-15T14:30:00Z"),
  updatedAt: ISODate("2024-01-15T14:30:00Z")
}

// Indexes
db.sales.createIndex({ userId: 1, saleDate: -1 })
db.sales.createIndex({ userId: 1, productId: 1, saleDate: -1 })
db.sales.createIndex({ userId: 1, customerId: 1, saleDate: -1 })
db.sales.createIndex({ saleDate: -1 })  // For time-series queries
```

#### 5.1.4 Customers Collection

```javascript
{
  _id: ObjectId("..."),
  userId: ObjectId("..."),
  customerId: "CUST-001",  // Business-friendly ID
  
  // Personal info
  name: "Jane Smith",
  email: "jane@example.com",
  phone: "+1234567890",
  
  // Address
  address: {
    street: "123 Main St",
    city: "New York",
    state: "NY",
    zipCode: "10001",
    country: "USA"
  },
  
  // RFM metrics (updated daily)
  rfm: {
    recency: 5,  // Days since last purchase
    frequency: 12,  // Total number of purchases
    monetary: 1250.00,  // Total spend
    rScore: 5,  // 1-5 scale
    fScore: 4,
    mScore: 5,
    rfmScore: 14,
    segment: "Champions",
    clv: 3500.00,  // Customer Lifetime Value
    updatedAt: ISODate("2024-01-15T02:00:00Z")
  },
  
  // Purchase history summary
  firstPurchaseDate: ISODate("2023-06-15"),
  lastPurchaseDate: ISODate("2024-01-10"),
  totalPurchases: 12,
  totalSpent: 1250.00,
  avgOrderValue: 104.17,
  
  // Preferences
  preferences: {
    favoriteCategories: ["Electronics", "Home & Garden"],
    communicationChannel: "email"
  },
  
  // Tags
  tags: ["vip", "frequent-buyer"],
  
  createdAt: ISODate("2023-06-15T00:00:00Z"),
  updatedAt: ISODate("2024-01-15T02:00:00Z")
}

// Indexes
db.customers.createIndex({ userId: 1, customerId: 1 }, { unique: true })
db.customers.createIndex({ userId: 1, email: 1 })
db.customers.createIndex({ userId: 1, "rfm.segment": 1 })
db.customers.createIndex({ userId: 1, lastPurchaseDate: -1 })
```

#### 5.1.5 Forecasts Collection

```javascript
{
  _id: ObjectId("..."),
  userId: ObjectId("..."),
  productId: ObjectId("..."),
  
  // Forecast metadata
  forecastDate: ISODate("2024-01-15"),  // Date forecast was generated
  forecastHorizon: 30,  // Days
  
  // Model info
  model: "ensemble",  // Model used
  confidence: 0.85,
  hasSeasonality: true,
  
  // Predictions
  predictions: [
    {
      date: ISODate("2024-01-16"),
      predictedDemand: 5.2,
      lowerBound: 3.1,
      upperBound: 7.3
    },
    {
      date: ISODate("2024-01-17"),
      predictedDemand: 5.5,
      lowerBound: 3.4,
      upperBound: 7.6
    }
    // ... 30 days
  ],
  
  // Summary statistics
  summary: {
    totalPredictedDemand: 156,
    avgDailyDemand: 5.2,
    peakDemandDate: ISODate("2024-01-25"),
    peakDemand: 8.5
  },
  
  // Training data info
  trainingDataPoints: 90,
  trainingStartDate: ISODate("2023-10-17"),
  trainingEndDate: ISODate("2024-01-14"),
  
  createdAt: ISODate("2024-01-15T02:00:00Z"),
  expiresAt: ISODate("2024-01-16T02:00:00Z")  // TTL index
}

// Indexes
db.forecasts.createIndex({ userId: 1, productId: 1, forecastDate: -1 })
db.forecasts.createIndex({ expiresAt: 1 }, { expireAfterSeconds: 0 })  // TTL
```

#### 5.1.6 PriceRecommendations Collection

```javascript
{
  _id: ObjectId("..."),
  userId: ObjectId("..."),
  productId: ObjectId("..."),
  
  // Current state
  currentPrice: 29.99,
  currentDemand: 5.2,
  
  // Recommendation
  recommendedPrice: 27.99,
  priceChangePct: -6.67,
  strategy: "profit",  // enum: ['profit', 'revenue', 'volume', 'clearance']
  
  // Predictions
  expectedDemand: 6.1,
  expectedRevenue: 170.69,
  expectedProfit: 79.32,
  
  // Model info
  priceElasticity: -1.2,
  confidence: 0.75,
  
  // Constraints used
  minPrice: 20.00,
  maxPrice: 50.00,
  
  // Metadata
  generatedAt: ISODate("2024-01-15T10:00:00Z"),
  validUntil: ISODate("2024-01-16T10:00:00Z"),
  
  // User action
  status: "pending",  // enum: ['pending', 'accepted', 'rejected', 'expired']
  appliedAt: null,
  appliedBy: null
}

// Indexes
db.priceRecommendations.createIndex({ userId: 1, productId: 1, generatedAt: -1 })
db.priceRecommendations.createIndex({ userId: 1, status: 1 })
```

### 5.2 Data Relationships

```
users (1) ──────< (N) products
users (1) ──────< (N) sales
users (1) ──────< (N) customers
products (1) ────< (N) sales
products (1) ────< (N) forecasts
products (1) ────< (N) priceRecommendations
customers (1) ───< (N) sales
```

### 5.3 Data Retention Policy

| Collection | Retention Period | Archive Strategy |
|------------|------------------|------------------|
| users | Indefinite | N/A |
| products | Indefinite | Soft delete (isActive=false) |
| sales | 3 years | Move to cold storage after 2 years |
| customers | Indefinite | Anonymize after 1 year of inactivity |
| forecasts | 7 days | Auto-delete via TTL index |
| priceRecommendations | 30 days | Auto-delete via TTL index |


## 6. API Design

### 6.1 API Versioning & Base URL

```
Base URL: https://api.retailcopilot.com/v1
Version: v1 (in URL path)
Content-Type: application/json
Authentication: Bearer token in Authorization header
```

### 6.2 Authentication Endpoints

#### POST /auth/register
Register a new user account

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "businessName": "ABC Retail Store",
  "firstName": "John",
  "lastName": "Doe"
}
```

**Response: 201 Created**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "65a1b2c3d4e5f6g7h8i9j0k1",
      "email": "user@example.com",
      "businessName": "ABC Retail Store",
      "role": "owner"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  },
  "message": "Registration successful"
}
```

#### POST /auth/login
Authenticate user and receive JWT token

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "65a1b2c3d4e5f6g7h8i9j0k1",
      "email": "user@example.com",
      "businessName": "ABC Retail Store",
      "role": "owner"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": "24h"
  }
}
```

#### POST /auth/refresh
Refresh JWT token

**Request:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": "24h"
  }
}
```

### 6.3 Product Endpoints

#### GET /products
Get all products with pagination and filtering

**Query Parameters:**
- `page` (number, default: 1)
- `limit` (number, default: 50, max: 100)
- `category` (string, optional)
- `search` (string, optional)
- `sortBy` (string, default: "createdAt")
- `sortOrder` (string, enum: ["asc", "desc"], default: "desc")
- `isActive` (boolean, optional)

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "products": [
      {
        "id": "65a1b2c3d4e5f6g7h8i9j0k1",
        "name": "Wireless Mouse",
        "sku": "WM-001",
        "category": "Electronics",
        "currentPrice": 29.99,
        "costPrice": 15.00,
        "stock": 150,
        "reorderLevel": 20,
        "profitMargin": 49.98,
        "isActive": true,
        "createdAt": "2024-01-01T00:00:00Z",
        "updatedAt": "2024-01-15T08:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 50,
      "total": 245,
      "pages": 5
    }
  }
}
```

#### GET /products/:id
Get single product by ID

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "id": "65a1b2c3d4e5f6g7h8i9j0k1",
    "name": "Wireless Mouse",
    "sku": "WM-001",
    "description": "Ergonomic wireless mouse",
    "category": "Electronics",
    "currentPrice": 29.99,
    "costPrice": 15.00,
    "stock": 150,
    "reorderLevel": 20,
    "priceHistory": [
      { "price": 34.99, "date": "2024-01-01T00:00:00Z" },
      { "price": 29.99, "date": "2024-01-15T00:00:00Z" }
    ],
    "analytics": {
      "totalSales": 450,
      "totalRevenue": 13497.00,
      "avgDailySales": 5.2,
      "inventoryTurnover": 3.0
    }
  }
}
```

#### POST /products
Create new product

**Request:**
```json
{
  "name": "Wireless Mouse",
  "sku": "WM-001",
  "category": "Electronics",
  "currentPrice": 29.99,
  "costPrice": 15.00,
  "stock": 150,
  "reorderLevel": 20,
  "description": "Ergonomic wireless mouse"
}
```

**Response: 201 Created**
```json
{
  "success": true,
  "data": {
    "id": "65a1b2c3d4e5f6g7h8i9j0k1",
    "name": "Wireless Mouse",
    "sku": "WM-001",
    "currentPrice": 29.99,
    "createdAt": "2024-01-15T10:00:00Z"
  },
  "message": "Product created successfully"
}
```

#### PUT /products/:id
Update product

**Request:**
```json
{
  "currentPrice": 27.99,
  "stock": 175
}
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "id": "65a1b2c3d4e5f6g7h8i9j0k1",
    "name": "Wireless Mouse",
    "currentPrice": 27.99,
    "stock": 175,
    "updatedAt": "2024-01-15T11:00:00Z"
  },
  "message": "Product updated successfully"
}
```

#### DELETE /products/:id
Delete product (soft delete)

**Response: 200 OK**
```json
{
  "success": true,
  "message": "Product deleted successfully"
}
```

#### POST /products/bulk-import
Bulk import products from CSV

**Request:** multipart/form-data
```
file: products.csv
```

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "imported": 150,
    "failed": 5,
    "errors": [
      { "row": 23, "error": "Duplicate SKU: WM-001" },
      { "row": 45, "error": "Invalid price format" }
    ]
  },
  "message": "Bulk import completed"
}
```

### 6.4 Sales Endpoints

#### GET /sales
Get sales transactions with filtering

**Query Parameters:**
- `page`, `limit` (pagination)
- `startDate`, `endDate` (date range)
- `productId` (filter by product)
- `customerId` (filter by customer)

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "sales": [
      {
        "id": "65a1b2c3d4e5f6g7h8i9j0k1",
        "productId": "65a1b2c3d4e5f6g7h8i9j0k2",
        "productName": "Wireless Mouse",
        "quantity": 2,
        "unitPrice": 29.99,
        "totalAmount": 59.98,
        "saleDate": "2024-01-15T14:30:00Z",
        "customerId": "CUST-001",
        "customerName": "Jane Smith"
      }
    ],
    "pagination": { "page": 1, "limit": 50, "total": 1250, "pages": 25 },
    "summary": {
      "totalSales": 1250,
      "totalRevenue": 45678.90,
      "avgOrderValue": 36.54
    }
  }
}
```

#### POST /sales
Record new sale

**Request:**
```json
{
  "productId": "65a1b2c3d4e5f6g7h8i9j0k1",
  "quantity": 2,
  "unitPrice": 29.99,
  "customerId": "CUST-001",
  "saleDate": "2024-01-15T14:30:00Z",
  "paymentMethod": "credit_card"
}
```

**Response: 201 Created**
```json
{
  "success": true,
  "data": {
    "id": "65a1b2c3d4e5f6g7h8i9j0k1",
    "totalAmount": 59.98,
    "saleDate": "2024-01-15T14:30:00Z"
  },
  "message": "Sale recorded successfully"
}
```

### 6.5 Analytics Endpoints

#### GET /analytics/dashboard
Get dashboard summary metrics

**Query Parameters:**
- `startDate`, `endDate` (date range)

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "revenue": {
      "current": 45678.90,
      "previous": 38234.50,
      "change": 19.46
    },
    "profit": {
      "current": 18234.50,
      "previous": 15123.20,
      "change": 20.58
    },
    "orders": {
      "current": 1250,
      "previous": 1100,
      "change": 13.64
    },
    "avgOrderValue": {
      "current": 36.54,
      "previous": 34.76,
      "change": 5.12
    },
    "topProducts": [
      {
        "productId": "65a1b2c3d4e5f6g7h8i9j0k1",
        "name": "Wireless Mouse",
        "revenue": 5678.90,
        "quantity": 189
      }
    ],
    "lowStockProducts": [
      {
        "productId": "65a1b2c3d4e5f6g7h8i9j0k2",
        "name": "USB Cable",
        "stock": 5,
        "reorderLevel": 20
      }
    ],
    "revenueByCategory": [
      { "category": "Electronics", "revenue": 25678.90 },
      { "category": "Accessories", "revenue": 12345.00 }
    ]
  }
}
```

#### GET /analytics/forecast/:productId
Get demand forecast for product

**Query Parameters:**
- `days` (number, default: 30)

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "productId": "65a1b2c3d4e5f6g7h8i9j0k1",
    "productName": "Wireless Mouse",
    "forecast": [
      {
        "date": "2024-01-16",
        "predictedDemand": 5.2,
        "lowerBound": 3.1,
        "upperBound": 7.3
      }
    ],
    "summary": {
      "totalPredictedDemand": 156,
      "avgDailyDemand": 5.2,
      "peakDemandDate": "2024-01-25",
      "peakDemand": 8.5
    },
    "confidence": 0.85,
    "model": "ensemble",
    "generatedAt": "2024-01-15T02:00:00Z"
  }
}
```

#### GET /analytics/price-recommendation/:productId
Get pricing recommendation

**Query Parameters:**
- `strategy` (enum: ['profit', 'revenue', 'volume', 'clearance'])

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "productId": "65a1b2c3d4e5f6g7h8i9j0k1",
    "currentPrice": 29.99,
    "recommendedPrice": 27.99,
    "priceChangePct": -6.67,
    "expectedDemand": 6.1,
    "expectedRevenue": 170.69,
    "expectedProfit": 79.32,
    "confidence": 0.75,
    "strategy": "profit",
    "reasoning": "Lower price will increase demand by 17%, resulting in 8% higher profit"
  }
}
```

#### GET /analytics/customer-segments
Get customer segmentation analysis

**Response: 200 OK**
```json
{
  "success": true,
  "data": {
    "segments": [
      {
        "segment": "Champions",
        "customerCount": 45,
        "totalRevenue": 67890.00,
        "avgClv": 3500.00,
        "characteristics": {
          "avgRecency": 8,
          "avgFrequency": 15,
          "avgMonetary": 1508.67
        }
      }
    ],
    "totalCustomers": 234,
    "totalClv": 456789.00
  }
}
```

### 6.6 Error Response Format

All errors follow consistent format:

**400 Bad Request**
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request data",
    "details": [
      { "field": "email", "message": "Invalid email format" },
      { "field": "price", "message": "Price must be greater than 0" }
    ]
  }
}
```

**401 Unauthorized**
```json
{
  "success": false,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Invalid or expired token"
  }
}
```

**404 Not Found**
```json
{
  "success": false,
  "error": {
    "code": "NOT_FOUND",
    "message": "Product not found"
  }
}
```

**500 Internal Server Error**
```json
{
  "success": false,
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An unexpected error occurred",
    "requestId": "req_abc123xyz"
  }
}
```

### 6.7 Rate Limiting

- **Authenticated requests**: 1000 requests per hour per user
- **Unauthenticated requests**: 100 requests per hour per IP
- **ML endpoints**: 100 requests per hour per user (computationally expensive)

**Rate Limit Headers:**
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 987
X-RateLimit-Reset: 1642348800
```

**429 Too Many Requests**
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please try again later.",
    "retryAfter": 3600
  }
}
```


## 7. ML Pipeline Design

### 7.1 ML Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     ML PIPELINE WORKFLOW                         │
└─────────────────────────────────────────────────────────────────┘

┌──────────────────┐
│  Data Collection │
│  - Sales history │
│  - Product data  │
│  - Price history │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Data Validation  │
│ - Check quality  │
│ - Min data req   │
│ - Outlier detect │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Preprocessing    │
│ - Time series    │
│ - Normalization  │
│ - Feature eng    │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Model Selection  │
│ - Seasonality?   │
│ - Data volume?   │
│ - Choose model   │
└────────┬─────────┘
         │
         ├──────────────┬──────────────┐
         ▼              ▼              ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ SARIMA Model │ │ Prophet Model│ │ Fallback MA  │
│ - Seasonal   │ │ - Robust     │ │ - Simple avg │
│ - Stationary │ │ - Holidays   │ │ - No training│
└──────┬───────┘ └──────┬───────┘ └──────┬───────┘
       │                │                │
       └────────────────┼────────────────┘
                        ▼
                ┌──────────────┐
                │   Ensemble   │
                │ Weighted avg │
                └──────┬───────┘
                       │
                       ▼
                ┌──────────────┐
                │ Post-process │
                │ - Bounds     │
                │ - Confidence │
                └──────┬───────┘
                       │
                       ▼
                ┌──────────────┐
                │ Cache Result │
                │ - Redis      │
                │ - MongoDB    │
                └──────────────┘
```

### 7.2 Data Requirements

#### Minimum Data Requirements

| Model Type | Minimum Data Points | Recommended | Optimal |
|------------|---------------------|-------------|---------|
| Demand Forecast | 30 days | 90 days | 365 days |
| Price Optimization | 10 price changes | 30 price changes | 60+ price changes |
| Customer Segmentation | 50 transactions | 200 transactions | 1000+ transactions |

#### Data Quality Checks

```python
# app/utils/data_validator.py

class DataValidator:
    """Validate data quality for ML models"""
    
    @staticmethod
    def validate_sales_data(sales_history):
        """
        Validate sales history for forecasting
        
        Returns: (is_valid, errors, warnings)
        """
        errors = []
        warnings = []
        
        # Check minimum data points
        if len(sales_history) < 30:
            errors.append("Insufficient data: need at least 30 days")
        
        # Check for missing dates
        dates = [s['date'] for s in sales_history]
        date_range = pd.date_range(min(dates), max(dates), freq='D')
        missing_dates = len(date_range) - len(dates)
        if missing_dates > len(dates) * 0.3:  # >30% missing
            warnings.append(f"High missing data: {missing_dates} days")
        
        # Check for outliers
        quantities = [s['quantity'] for s in sales_history]
        q1, q3 = np.percentile(quantities, [25, 75])
        iqr = q3 - q1
        outliers = [q for q in quantities if q > q3 + 3*iqr or q < q1 - 3*iqr]
        if len(outliers) > len(quantities) * 0.05:  # >5% outliers
            warnings.append(f"High outlier count: {len(outliers)}")
        
        # Check for zero variance
        if np.std(quantities) < 0.1:
            warnings.append("Very low variance in sales data")
        
        is_valid = len(errors) == 0
        return is_valid, errors, warnings
```

### 7.3 Feature Engineering

#### Time Series Features

```python
def create_time_features(df):
    """
    Create time-based features for forecasting
    """
    df['day_of_week'] = df.index.dayofweek
    df['day_of_month'] = df.index.day
    df['week_of_year'] = df.index.isocalendar().week
    df['month'] = df.index.month
    df['quarter'] = df.index.quarter
    df['is_weekend'] = df['day_of_week'].isin([5, 6]).astype(int)
    df['is_month_start'] = df.index.is_month_start.astype(int)
    df['is_month_end'] = df.index.is_month_end.astype(int)
    
    return df

def create_lag_features(df, lags=[1, 7, 14, 30]):
    """
    Create lagged features for time series
    """
    for lag in lags:
        df[f'lag_{lag}'] = df['quantity'].shift(lag)
    
    return df

def create_rolling_features(df, windows=[7, 14, 30]):
    """
    Create rolling statistics features
    """
    for window in windows:
        df[f'rolling_mean_{window}'] = df['quantity'].rolling(window).mean()
        df[f'rolling_std_{window}'] = df['quantity'].rolling(window).std()
        df[f'rolling_min_{window}'] = df['quantity'].rolling(window).min()
        df[f'rolling_max_{window}'] = df['quantity'].rolling(window).max()
    
    return df
```

### 7.4 Model Training Pipeline

#### Automated Retraining Strategy

```python
# app/jobs/model_trainer.py

class ModelTrainer:
    """
    Automated model training and retraining
    """
    
    def should_retrain(self, product_id, last_training_date):
        """
        Determine if model needs retraining
        
        Retrain if:
        - More than 7 days since last training
        - Significant new data (>20% increase)
        - Model performance degraded (>10% error increase)
        """
        days_since_training = (datetime.now() - last_training_date).days
        
        if days_since_training > 7:
            return True, "Scheduled retraining"
        
        # Check new data volume
        new_data_count = self.get_new_data_count(product_id, last_training_date)
        total_data_count = self.get_total_data_count(product_id)
        
        if new_data_count / total_data_count > 0.2:
            return True, "Significant new data"
        
        # Check model performance
        recent_error = self.calculate_recent_error(product_id)
        baseline_error = self.get_baseline_error(product_id)
        
        if recent_error > baseline_error * 1.1:
            return True, "Performance degradation"
        
        return False, "No retraining needed"
    
    def train_all_models(self):
        """
        Batch training for all products (daily job)
        """
        products = self.get_active_products()
        
        for product in products:
            try:
                if self.should_retrain(product['id'], product['last_training_date']):
                    self.train_product_model(product['id'])
                    logger.info(f"Retrained model for product {product['id']}")
            except Exception as e:
                logger.error(f"Training failed for {product['id']}: {e}")
```

### 7.5 Model Evaluation Metrics

#### Forecast Accuracy Metrics

```python
def evaluate_forecast(actual, predicted):
    """
    Calculate forecast accuracy metrics
    """
    # Mean Absolute Error
    mae = np.mean(np.abs(actual - predicted))
    
    # Mean Absolute Percentage Error
    mape = np.mean(np.abs((actual - predicted) / actual)) * 100
    
    # Root Mean Squared Error
    rmse = np.sqrt(np.mean((actual - predicted) ** 2))
    
    # R-squared
    ss_res = np.sum((actual - predicted) ** 2)
    ss_tot = np.sum((actual - np.mean(actual)) ** 2)
    r2 = 1 - (ss_res / ss_tot)
    
    return {
        'mae': mae,
        'mape': mape,
        'rmse': rmse,
        'r2': r2,
        'accuracy': 100 - mape  # Simple accuracy metric
    }
```

### 7.6 Model Versioning & Storage

```python
# app/utils/model_manager.py

class ModelManager:
    """
    Manage ML model lifecycle
    """
    
    def __init__(self, storage_path='models/'):
        self.storage_path = storage_path
    
    def save_model(self, model, product_id, version, metadata):
        """
        Save model with versioning
        """
        model_path = f"{self.storage_path}/{product_id}_v{version}.pkl"
        
        # Save model
        joblib.dump(model, model_path)
        
        # Save metadata
        metadata_path = f"{self.storage_path}/{product_id}_v{version}_meta.json"
        with open(metadata_path, 'w') as f:
            json.dump({
                'product_id': product_id,
                'version': version,
                'trained_at': datetime.now().isoformat(),
                'training_data_points': metadata['data_points'],
                'performance_metrics': metadata['metrics'],
                'model_type': metadata['model_type']
            }, f)
        
        logger.info(f"Saved model {product_id} v{version}")
    
    def load_model(self, product_id, version='latest'):
        """
        Load model by version
        """
        if version == 'latest':
            version = self.get_latest_version(product_id)
        
        model_path = f"{self.storage_path}/{product_id}_v{version}.pkl"
        
        if not os.path.exists(model_path):
            raise FileNotFoundError(f"Model not found: {model_path}")
        
        return joblib.load(model_path)
    
    def get_latest_version(self, product_id):
        """
        Get latest model version for product
        """
        files = glob.glob(f"{self.storage_path}/{product_id}_v*.pkl")
        if not files:
            return None
        
        versions = [int(f.split('_v')[1].split('.')[0]) for f in files]
        return max(versions)
```

### 7.7 Prediction Caching Strategy

```python
# app/services/forecast_service.py

class ForecastService:
    """
    Forecast service with intelligent caching
    """
    
    def get_forecast(self, product_id, days=30):
        """
        Get forecast with cache-first strategy
        """
        # Check cache
        cache_key = f"forecast:{product_id}:{date.today()}"
        cached = redis_client.get(cache_key)
        
        if cached:
            logger.info(f"Cache hit for {product_id}")
            return json.loads(cached)
        
        # Generate new forecast
        logger.info(f"Cache miss for {product_id}, generating forecast")
        
        sales_data = self.get_sales_history(product_id)
        forecaster = DemandForecaster()
        result = forecaster.predict(sales_data, days)
        
        # Cache for 24 hours
        redis_client.setex(
            cache_key,
            86400,  # 24 hours
            json.dumps(result)
        )
        
        # Also save to MongoDB for historical tracking
        self.save_forecast_to_db(product_id, result)
        
        return result
```

### 7.8 A/B Testing Framework

```python
# app/models/ab_tester.py

class PricingABTest:
    """
    A/B testing framework for pricing strategies
    """
    
    def create_test(self, product_id, control_price, test_price, duration_days=14):
        """
        Create new A/B test for pricing
        """
        test = {
            'product_id': product_id,
            'control_price': control_price,
            'test_price': test_price,
            'start_date': datetime.now(),
            'end_date': datetime.now() + timedelta(days=duration_days),
            'status': 'active',
            'results': {
                'control': {'sales': 0, 'revenue': 0},
                'test': {'sales': 0, 'revenue': 0}
            }
        }
        
        return db.ab_tests.insert_one(test)
    
    def analyze_results(self, test_id):
        """
        Statistical analysis of A/B test results
        """
        test = db.ab_tests.find_one({'_id': test_id})
        
        control = test['results']['control']
        test_group = test['results']['test']
        
        # Calculate conversion rates
        control_rate = control['sales'] / control['impressions']
        test_rate = test_group['sales'] / test_group['impressions']
        
        # Statistical significance (Chi-square test)
        from scipy.stats import chi2_contingency
        
        contingency_table = [
            [control['sales'], control['impressions'] - control['sales']],
            [test_group['sales'], test_group['impressions'] - test_group['sales']]
        ]
        
        chi2, p_value, dof, expected = chi2_contingency(contingency_table)
        
        is_significant = p_value < 0.05
        
        return {
            'control_conversion': control_rate,
            'test_conversion': test_rate,
            'lift': (test_rate - control_rate) / control_rate * 100,
            'p_value': p_value,
            'is_significant': is_significant,
            'recommendation': 'adopt_test' if is_significant and test_rate > control_rate else 'keep_control'
        }
```


## 8. Deployment Architecture

### 8.1 Cloud Infrastructure (AWS)

```
┌─────────────────────────────────────────────────────────────────┐
│                         AWS CLOUD                                │
│                                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                    Route 53 (DNS)                           │ │
│  │  - retailcopilot.com → CloudFront                          │ │
│  │  - api.retailcopilot.com → ALB                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                    │
│         ┌────────────────────┴────────────────────┐              │
│         ▼                                         ▼              │
│  ┌──────────────────┐                    ┌──────────────────┐   │
│  │   CloudFront     │                    │  Application     │   │
│  │   (CDN)          │                    │  Load Balancer   │   │
│  │  - Static assets │                    │  - SSL/TLS       │   │
│  │  - React build   │                    │  - Health checks │   │
│  └──────────────────┘                    └────────┬─────────┘   │
│         │                                          │              │
│         ▼                                          │              │
│  ┌──────────────────┐                             │              │
│  │   S3 Bucket      │                             │              │
│  │  - Frontend      │                             │              │
│  │  - CSV uploads   │                             │              │
│  │  - Reports       │                             │              │
│  └──────────────────┘                             │              │
│                                                    │              │
│                    ┌───────────────────────────────┤              │
│                    │                               │              │
│         ┌──────────▼──────────┐         ┌─────────▼──────────┐  │
│         │  ECS Cluster 1      │         │  ECS Cluster 2     │  │
│         │  (API Servers)      │         │  (ML Service)      │  │
│         │                     │         │                    │  │
│         │  ┌──────────────┐  │         │  ┌──────────────┐ │  │
│         │  │ Node.js API  │  │         │  │ Flask ML     │ │  │
│         │  │ Container 1  │  │         │  │ Container 1  │ │  │
│         │  └──────────────┘  │         │  └──────────────┘ │  │
│         │  ┌──────────────┐  │         │  ┌──────────────┐ │  │
│         │  │ Node.js API  │  │         │  │ Flask ML     │ │  │
│         │  │ Container 2  │  │         │  │ Container 2  │ │  │
│         │  └──────────────┘  │         │  └──────────────┘ │  │
│         │  ┌──────────────┐  │         │                    │  │
│         │  │ Node.js API  │  │         │  Auto-scaling:     │  │
│         │  │ Container 3  │  │         │  Min: 2, Max: 10   │  │
│         │  └──────────────┘  │         └────────────────────┘  │
│         │                     │                  │               │
│         │  Auto-scaling:      │                  │               │
│         │  Min: 3, Max: 20    │                  │               │
│         └─────────────────────┘                  │               │
│                    │                             │               │
│                    └──────────┬──────────────────┘               │
│                               │                                  │
│                    ┌──────────▼──────────┐                       │
│                    │  ElastiCache        │                       │
│                    │  (Redis)            │                       │
│                    │  - Session cache    │                       │
│                    │  - API cache        │                       │
│                    │  - Rate limiting    │                       │
│                    └─────────────────────┘                       │
│                               │                                  │
│                    ┌──────────▼──────────┐                       │
│                    │  MongoDB Atlas      │                       │
│                    │  (Managed)          │                       │
│                    │  - Primary DB       │                       │
│                    │  - Auto-backup      │                       │
│                    │  - Multi-AZ         │                       │
│                    └─────────────────────┘                       │
│                                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                    Supporting Services                      │ │
│  │                                                              │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │ │
│  │  │ CloudWatch   │  │ Secrets      │  │ SES (Email)  │    │ │
│  │  │ - Logs       │  │ Manager      │  │ - Alerts     │    │ │
│  │  │ - Metrics    │  │ - API keys   │  │ - Reports    │    │ │
│  │  │ - Alarms     │  │ - DB creds   │  │              │    │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘    │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### 8.2 Container Configuration

#### API Server Dockerfile

```dockerfile
# server/Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Production image
FROM node:18-alpine

WORKDIR /app

# Copy from builder
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/src ./src
COPY --from=builder /app/package.json ./

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

USER nodejs

EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD node -e "require('http').get('http://localhost:5000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

CMD ["node", "src/index.js"]
```

#### ML Service Dockerfile

```dockerfile
# ml-service/Dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user
RUN useradd -m -u 1001 mluser && \
    chown -R mluser:mluser /app

USER mluser

EXPOSE 5001

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s \
  CMD python -c "import requests; requests.get('http://localhost:5001/health')"

CMD ["gunicorn", "--bind", "0.0.0.0:5001", "--workers", "4", "--timeout", "120", "app:create_app()"]
```

### 8.3 Docker Compose (Local Development)

```yaml
# docker-compose.yml
version: '3.8'

services:
  mongodb:
    image: mongo:5.0
    container_name: retail-mongodb
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password
    volumes:
      - mongodb_data:/data/db
    networks:
      - retail-network

  redis:
    image: redis:7-alpine
    container_name: retail-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - retail-network

  api:
    build:
      context: ./server
      dockerfile: Dockerfile
    container_name: retail-api
    ports:
      - "5000:5000"
    environment:
      NODE_ENV: development
      PORT: 5000
      MONGODB_URI: mongodb://admin:password@mongodb:27017/retail-copilot?authSource=admin
      REDIS_URL: redis://redis:6379
      ML_SERVICE_URL: http://ml-service:5001
      JWT_SECRET: dev-secret-key
    depends_on:
      - mongodb
      - redis
    volumes:
      - ./server/src:/app/src
    networks:
      - retail-network

  ml-service:
    build:
      context: ./ml-service
      dockerfile: Dockerfile
    container_name: retail-ml
    ports:
      - "5001:5001"
    environment:
      FLASK_ENV: development
      MODEL_STORAGE_PATH: /app/models
    volumes:
      - ./ml-service/app:/app/app
      - ml_models:/app/models
    networks:
      - retail-network

  client:
    build:
      context: ./client
      dockerfile: Dockerfile.dev
    container_name: retail-client
    ports:
      - "3000:3000"
    environment:
      VITE_API_URL: http://localhost:5000/api
    volumes:
      - ./client/src:/app/src
    networks:
      - retail-network

volumes:
  mongodb_data:
  redis_data:
  ml_models:

networks:
  retail-network:
    driver: bridge
```

### 8.4 Kubernetes Deployment (Production)

#### API Deployment

```yaml
# k8s/api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: retail-api
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: retail-api
  template:
    metadata:
      labels:
        app: retail-api
    spec:
      containers:
      - name: api
        image: retailcopilot/api:latest
        ports:
        - containerPort: 5000
        env:
        - name: NODE_ENV
          value: "production"
        - name: MONGODB_URI
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: mongodb-uri
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: api-secrets
              key: jwt-secret
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 5000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 5000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: retail-api-service
  namespace: production
spec:
  selector:
    app: retail-api
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5000
  type: LoadBalancer
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: retail-api-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: retail-api
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### 8.5 CI/CD Pipeline (GitHub Actions)

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: |
          cd server && npm ci
          cd ../client && npm ci
      
      - name: Run tests
        run: |
          cd server && npm test
          cd ../client && npm test
      
      - name: Run linting
        run: |
          cd server && npm run lint
          cd ../client && npm run lint

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1
      
      - name: Build and push API image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          cd server
          docker build -t $ECR_REGISTRY/retail-api:$IMAGE_TAG .
          docker push $ECR_REGISTRY/retail-api:$IMAGE_TAG
      
      - name: Build and push ML service image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          cd ml-service
          docker build -t $ECR_REGISTRY/retail-ml:$IMAGE_TAG .
          docker push $ECR_REGISTRY/retail-ml:$IMAGE_TAG
      
      - name: Build and deploy frontend
        run: |
          cd client
          npm run build
          aws s3 sync dist/ s3://retailcopilot-frontend --delete
          aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_ID }} --paths "/*"

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to ECS
        run: |
          aws ecs update-service --cluster retail-cluster --service retail-api --force-new-deployment
          aws ecs update-service --cluster retail-cluster --service retail-ml --force-new-deployment
```

### 8.6 Environment Configuration

```bash
# Production environment variables
# Stored in AWS Secrets Manager

# API Server
NODE_ENV=production
PORT=5000
MONGODB_URI=mongodb+srv://user:pass@cluster.mongodb.net/retail
REDIS_URL=redis://retail-cache.abc123.ng.0001.use1.cache.amazonaws.com:6379
ML_SERVICE_URL=http://ml-service.internal:5001
JWT_SECRET=<strong-random-secret>
JWT_EXPIRES_IN=24h
CORS_ORIGIN=https://app.retailcopilot.com
RATE_LIMIT_WINDOW=3600000
RATE_LIMIT_MAX=1000

# ML Service
FLASK_ENV=production
MODEL_STORAGE_PATH=/app/models
MAX_WORKERS=4
WORKER_TIMEOUT=120

# AWS Services
AWS_REGION=us-east-1
S3_BUCKET=retailcopilot-uploads
SES_FROM_EMAIL=noreply@retailcopilot.com

# Monitoring
SENTRY_DSN=https://...@sentry.io/...
LOG_LEVEL=info
```


## 9. Security Architecture

### 9.1 Security Layers

```
┌─────────────────────────────────────────────────────────────┐
│                    SECURITY LAYERS                           │
└─────────────────────────────────────────────────────────────┘

Layer 1: Network Security
├── CloudFront WAF (Web Application Firewall)
│   ├── SQL injection protection
│   ├── XSS protection
│   ├── Rate limiting
│   └── Geo-blocking
├── VPC (Virtual Private Cloud)
│   ├── Private subnets for databases
│   ├── Security groups
│   └── Network ACLs
└── DDoS Protection (AWS Shield)

Layer 2: Application Security
├── HTTPS/TLS 1.3 (SSL certificates)
├── CORS policy (whitelist origins)
├── Helmet.js (security headers)
│   ├── X-Frame-Options: DENY
│   ├── X-Content-Type-Options: nosniff
│   ├── Strict-Transport-Security
│   └── Content-Security-Policy
├── Rate limiting (express-rate-limit)
└── Input validation (express-validator)

Layer 3: Authentication & Authorization
├── JWT tokens (signed with HS256)
├── Password hashing (bcrypt, 10 rounds)
├── Role-based access control (RBAC)
├── Token expiration (24 hours)
└── Refresh token mechanism

Layer 4: Data Security
├── Encryption at rest (MongoDB encryption)
├── Encryption in transit (TLS)
├── Sensitive data masking in logs
├── PII data handling compliance
└── Database access controls

Layer 5: Monitoring & Auditing
├── Security event logging
├── Failed login attempt tracking
├── API access logs
├── Anomaly detection
└── Regular security audits
```

### 9.2 Authentication Implementation

#### JWT Token Structure

```javascript
// Token payload
{
  "userId": "65a1b2c3d4e5f6g7h8i9j0k1",
  "email": "user@example.com",
  "role": "owner",
  "iat": 1642348800,  // Issued at
  "exp": 1642435200   // Expires at (24 hours)
}

// Token generation
import jwt from 'jsonwebtoken';

const generateToken = (user) => {
  return jwt.sign(
    {
      userId: user._id,
      email: user.email,
      role: user.role
    },
    process.env.JWT_SECRET,
    {
      expiresIn: '24h',
      algorithm: 'HS256'
    }
  );
};

// Token verification middleware
export const authenticate = (req, res, next) => {
  try {
    const token = req.header('Authorization')?.replace('Bearer ', '');
    
    if (!token) {
      return res.status(401).json({
        success: false,
        error: { code: 'NO_TOKEN', message: 'Authentication required' }
      });
    }

    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // Check token expiration
    if (decoded.exp < Date.now() / 1000) {
      return res.status(401).json({
        success: false,
        error: { code: 'TOKEN_EXPIRED', message: 'Token has expired' }
      });
    }

    req.userId = decoded.userId;
    req.userRole = decoded.role;
    next();
  } catch (error) {
    return res.status(401).json({
      success: false,
      error: { code: 'INVALID_TOKEN', message: 'Invalid token' }
    });
  }
};
```

#### Password Security

```javascript
// Password hashing
import bcrypt from 'bcryptjs';

const hashPassword = async (password) => {
  const salt = await bcrypt.genSalt(10);
  return bcrypt.hash(password, salt);
};

const comparePassword = async (password, hash) => {
  return bcrypt.compare(password, hash);
};

// Password strength validation
const validatePasswordStrength = (password) => {
  const minLength = 8;
  const hasUpperCase = /[A-Z]/.test(password);
  const hasLowerCase = /[a-z]/.test(password);
  const hasNumbers = /\d/.test(password);
  const hasSpecialChar = /[!@#$%^&*(),.?":{}|<>]/.test(password);

  const errors = [];
  
  if (password.length < minLength) {
    errors.push(`Password must be at least ${minLength} characters`);
  }
  if (!hasUpperCase) {
    errors.push('Password must contain uppercase letter');
  }
  if (!hasLowerCase) {
    errors.push('Password must contain lowercase letter');
  }
  if (!hasNumbers) {
    errors.push('Password must contain number');
  }
  if (!hasSpecialChar) {
    errors.push('Password must contain special character');
  }

  return {
    isValid: errors.length === 0,
    errors
  };
};
```

### 9.3 Authorization (RBAC)

```javascript
// Role-based access control middleware
export const authorize = (...allowedRoles) => {
  return (req, res, next) => {
    if (!req.userRole) {
      return res.status(401).json({
        success: false,
        error: { code: 'UNAUTHORIZED', message: 'Authentication required' }
      });
    }

    if (!allowedRoles.includes(req.userRole)) {
      return res.status(403).json({
        success: false,
        error: { code: 'FORBIDDEN', message: 'Insufficient permissions' }
      });
    }

    next();
  };
};

// Usage in routes
router.delete('/products/:id', 
  authenticate, 
  authorize('owner'),  // Only owners can delete
  productController.remove
);

router.get('/products', 
  authenticate, 
  authorize('owner', 'manager'),  // Both can view
  productController.getAll
);
```

### 9.4 Input Validation & Sanitization

```javascript
// Input validation middleware
import { body, param, query, validationResult } from 'express-validator';

export const validateProduct = [
  body('name')
    .trim()
    .notEmpty().withMessage('Product name is required')
    .isLength({ max: 200 }).withMessage('Name too long')
    .escape(),  // Prevent XSS
  
  body('sku')
    .trim()
    .notEmpty().withMessage('SKU is required')
    .matches(/^[A-Z0-9-]+$/).withMessage('Invalid SKU format')
    .escape(),
  
  body('currentPrice')
    .isFloat({ min: 0 }).withMessage('Price must be positive number'),
  
  body('stock')
    .optional()
    .isInt({ min: 0 }).withMessage('Stock must be non-negative integer'),
  
  body('category')
    .optional()
    .trim()
    .escape(),
  
  // Validation result handler
  (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({
        success: false,
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Invalid input data',
          details: errors.array()
        }
      });
    }
    next();
  }
];

// SQL injection prevention (MongoDB)
// MongoDB automatically escapes queries, but still validate input
const sanitizeMongoQuery = (query) => {
  // Remove MongoDB operators from user input
  const sanitized = {};
  for (const [key, value] of Object.entries(query)) {
    if (typeof value === 'string' && !key.startsWith('$')) {
      sanitized[key] = value;
    }
  }
  return sanitized;
};
```

### 9.5 Rate Limiting

```javascript
// Rate limiting configuration
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';
import redis from './config/redis.js';

// General API rate limit
export const apiLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rl:api:'
  }),
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 1000, // 1000 requests per hour
  message: {
    success: false,
    error: {
      code: 'RATE_LIMIT_EXCEEDED',
      message: 'Too many requests, please try again later'
    }
  },
  standardHeaders: true,
  legacyHeaders: false,
  keyGenerator: (req) => {
    // Use userId if authenticated, otherwise IP
    return req.userId || req.ip;
  }
});

// Stricter limit for authentication endpoints
export const authLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rl:auth:'
  }),
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts per 15 minutes
  skipSuccessfulRequests: true, // Don't count successful logins
  message: {
    success: false,
    error: {
      code: 'TOO_MANY_ATTEMPTS',
      message: 'Too many login attempts, please try again later'
    }
  }
});

// ML endpoint rate limit (expensive operations)
export const mlLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rl:ml:'
  }),
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 100, // 100 ML requests per hour
  message: {
    success: false,
    error: {
      code: 'ML_RATE_LIMIT',
      message: 'ML request limit exceeded'
    }
  }
});

// Apply to routes
app.use('/api/', apiLimiter);
app.use('/api/auth/login', authLimiter);
app.use('/api/analytics/forecast', mlLimiter);
```

### 9.6 Security Headers

```javascript
// Security headers middleware
import helmet from 'helmet';

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", "data:", "https:"],
      connectSrc: ["'self'", "https://api.retailcopilot.com"],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      mediaSrc: ["'self'"],
      frameSrc: ["'none'"]
    }
  },
  hsts: {
    maxAge: 31536000, // 1 year
    includeSubDomains: true,
    preload: true
  },
  noSniff: true,
  xssFilter: true,
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' }
}));

// CORS configuration
import cors from 'cors';

const corsOptions = {
  origin: (origin, callback) => {
    const allowedOrigins = [
      'https://app.retailcopilot.com',
      'https://retailcopilot.com'
    ];
    
    if (process.env.NODE_ENV === 'development') {
      allowedOrigins.push('http://localhost:3000');
    }
    
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
  optionsSuccessStatus: 200
};

app.use(cors(corsOptions));
```

### 9.7 Sensitive Data Protection

```javascript
// Logging with sensitive data masking
import winston from 'winston';

const maskSensitiveData = winston.format((info) => {
  const sensitiveFields = ['password', 'token', 'apiKey', 'secret'];
  
  const mask = (obj) => {
    if (typeof obj !== 'object' || obj === null) return obj;
    
    const masked = { ...obj };
    for (const key in masked) {
      if (sensitiveFields.some(field => key.toLowerCase().includes(field))) {
        masked[key] = '***REDACTED***';
      } else if (typeof masked[key] === 'object') {
        masked[key] = mask(masked[key]);
      }
    }
    return masked;
  };
  
  info.message = typeof info.message === 'object' 
    ? mask(info.message) 
    : info.message;
  
  return info;
});

export const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    maskSensitiveData(),
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

// Database query logging (exclude sensitive fields)
mongoose.set('debug', (collectionName, method, query, doc) => {
  const sanitizedQuery = { ...query };
  delete sanitizedQuery.password;
  
  logger.debug({
    type: 'mongodb',
    collection: collectionName,
    method,
    query: sanitizedQuery
  });
});
```

### 9.8 Security Checklist

**Pre-Deployment Security Audit:**

- [ ] All secrets stored in AWS Secrets Manager (not in code)
- [ ] HTTPS enforced on all endpoints
- [ ] JWT secret is strong and rotated regularly
- [ ] Database credentials use principle of least privilege
- [ ] Rate limiting configured on all public endpoints
- [ ] Input validation on all user inputs
- [ ] SQL/NoSQL injection prevention verified
- [ ] XSS protection enabled
- [ ] CSRF protection for state-changing operations
- [ ] Security headers configured (Helmet.js)
- [ ] CORS properly configured
- [ ] File upload validation and size limits
- [ ] Error messages don't leak sensitive information
- [ ] Logging excludes sensitive data
- [ ] Dependencies scanned for vulnerabilities (npm audit)
- [ ] Container images scanned for vulnerabilities
- [ ] Backup and disaster recovery tested
- [ ] Incident response plan documented
- [ ] Security monitoring and alerting configured
- [ ] Penetration testing completed
- [ ] GDPR/privacy compliance verified


## 10. Scalability & Performance

### 10.1 Scalability Strategy

#### Horizontal Scaling

```
┌─────────────────────────────────────────────────────────────┐
│                  HORIZONTAL SCALING                          │
└─────────────────────────────────────────────────────────────┘

Load Distribution:
                    ┌──────────────────┐
                    │  Load Balancer   │
                    │  (ALB)           │
                    └────────┬─────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ API Server 1 │     │ API Server 2 │     │ API Server N │
│ (Stateless)  │     │ (Stateless)  │     │ (Stateless)  │
└──────────────┘     └──────────────┘     └──────────────┘

Auto-scaling Triggers:
- CPU utilization > 70%
- Memory utilization > 80%
- Request queue depth > 100
- Response time > 500ms (p95)

Scaling Policies:
- Scale up: Add 2 instances when threshold exceeded for 2 minutes
- Scale down: Remove 1 instance when below 40% for 10 minutes
- Min instances: 3 (high availability)
- Max instances: 20 (cost control)
```

#### Database Scaling

```javascript
// MongoDB connection with replica set
const mongoOptions = {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  maxPoolSize: 50,  // Connection pool
  minPoolSize: 10,
  serverSelectionTimeoutMS: 5000,
  socketTimeoutMS: 45000,
  // Read preference for scaling reads
  readPreference: 'secondaryPreferred',
  // Write concern for consistency
  w: 'majority',
  wtimeoutMS: 5000
};

// Sharding strategy for large datasets
// Shard key: userId (ensures user data stays together)
db.sales.createIndex({ userId: 1, saleDate: -1 });
sh.shardCollection("retail.sales", { userId: 1 });

// Time-series collections for sales data (MongoDB 5.0+)
db.createCollection("sales_timeseries", {
  timeseries: {
    timeField: "saleDate",
    metaField: "userId",
    granularity: "hours"
  }
});
```

### 10.2 Caching Strategy

#### Multi-Level Caching

```javascript
// Cache hierarchy
class CacheManager {
  constructor() {
    this.l1Cache = new Map();  // In-memory (fast, small)
    this.l2Cache = redis;       // Redis (fast, medium)
    this.l3Cache = mongodb;     // Database (slow, large)
  }

  async get(key) {
    // L1: In-memory cache (fastest)
    if (this.l1Cache.has(key)) {
      return this.l1Cache.get(key);
    }

    // L2: Redis cache
    const redisValue = await this.l2Cache.get(key);
    if (redisValue) {
      // Populate L1 cache
      this.l1Cache.set(key, JSON.parse(redisValue));
      return JSON.parse(redisValue);
    }

    // L3: Database
    const dbValue = await this.l3Cache.findOne({ key });
    if (dbValue) {
      // Populate L2 and L1
      await this.l2Cache.setex(key, 3600, JSON.stringify(dbValue.value));
      this.l1Cache.set(key, dbValue.value);
      return dbValue.value;
    }

    return null;
  }

  async set(key, value, ttl = 3600) {
    // Set in all levels
    this.l1Cache.set(key, value);
    await this.l2Cache.setex(key, ttl, JSON.stringify(value));
    // L3 is source of truth, updated separately
  }

  async invalidate(key) {
    this.l1Cache.delete(key);
    await this.l2Cache.del(key);
  }
}

// Cache warming strategy
class CacheWarmer {
  async warmCache() {
    // Pre-populate cache with frequently accessed data
    const popularProducts = await Product.find()
      .sort({ 'analytics.totalSales': -1 })
      .limit(100);

    for (const product of popularProducts) {
      const cacheKey = `product:${product._id}`;
      await cacheManager.set(cacheKey, product, 7200);
    }

    logger.info('Cache warming completed');
  }

  // Run daily at 2 AM
  scheduleWarming() {
    cron.schedule('0 2 * * *', () => {
      this.warmCache();
    });
  }
}
```

#### Cache Invalidation Patterns

```javascript
// Event-driven cache invalidation
class CacheInvalidator {
  constructor() {
    this.setupEventListeners();
  }

  setupEventListeners() {
    // Product updated
    eventEmitter.on('product:updated', async (productId) => {
      await this.invalidateProduct(productId);
    });

    // Sale recorded
    eventEmitter.on('sale:created', async (saleData) => {
      await this.invalidateProductAnalytics(saleData.productId);
      await this.invalidateDashboard(saleData.userId);
    });

    // Price changed
    eventEmitter.on('price:changed', async (productId) => {
      await this.invalidatePriceRecommendations(productId);
      await this.invalidateForecast(productId);
    });
  }

  async invalidateProduct(productId) {
    const keys = [
      `product:${productId}`,
      `products:list:*`,  // Wildcard for all product lists
      `analytics:product:${productId}`
    ];
    
    for (const key of keys) {
      if (key.includes('*')) {
        // Delete all matching keys
        const matchingKeys = await redis.keys(key);
        if (matchingKeys.length > 0) {
          await redis.del(...matchingKeys);
        }
      } else {
        await redis.del(key);
      }
    }
  }
}
```

### 10.3 Database Optimization

#### Indexing Strategy

```javascript
// Compound indexes for common queries
db.products.createIndex({ userId: 1, category: 1, isActive: 1 });
db.products.createIndex({ userId: 1, stock: 1 });  // Low stock queries
db.products.createIndex({ userId: 1, 'analytics.totalSales': -1 });  // Top products

// Time-series indexes
db.sales.createIndex({ userId: 1, saleDate: -1 });
db.sales.createIndex({ userId: 1, productId: 1, saleDate: -1 });

// Text search index
db.products.createIndex({ 
  name: "text", 
  description: "text", 
  sku: "text" 
}, {
  weights: {
    name: 10,
    sku: 5,
    description: 1
  }
});

// TTL index for auto-deletion
db.forecasts.createIndex({ expiresAt: 1 }, { expireAfterSeconds: 0 });

// Analyze index usage
db.products.aggregate([
  { $indexStats: {} }
]);
```

#### Query Optimization

```javascript
// Efficient pagination with cursor-based approach
class PaginationService {
  async getCursorPaginated(model, query, options) {
    const { limit = 50, cursor, sortField = 'createdAt' } = options;

    // Build query with cursor
    const finalQuery = { ...query };
    if (cursor) {
      finalQuery[sortField] = { $lt: cursor };
    }

    // Fetch one extra to determine if there are more results
    const results = await model
      .find(finalQuery)
      .sort({ [sortField]: -1 })
      .limit(limit + 1)
      .lean();  // Return plain objects (faster)

    const hasMore = results.length > limit;
    const items = hasMore ? results.slice(0, -1) : results;
    const nextCursor = hasMore ? items[items.length - 1][sortField] : null;

    return {
      items,
      nextCursor,
      hasMore
    };
  }
}

// Aggregation pipeline optimization
const getTopProducts = async (userId, days = 30) => {
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - days);

  return await Sale.aggregate([
    // Stage 1: Filter by user and date
    {
      $match: {
        userId: new ObjectId(userId),
        saleDate: { $gte: startDate }
      }
    },
    // Stage 2: Group by product
    {
      $group: {
        _id: '$productId',
        totalQuantity: { $sum: '$quantity' },
        totalRevenue: { $sum: '$totalAmount' },
        salesCount: { $sum: 1 }
      }
    },
    // Stage 3: Sort by revenue
    {
      $sort: { totalRevenue: -1 }
    },
    // Stage 4: Limit results
    {
      $limit: 10
    },
    // Stage 5: Lookup product details
    {
      $lookup: {
        from: 'products',
        localField: '_id',
        foreignField: '_id',
        as: 'product'
      }
    },
    // Stage 6: Unwind and project
    {
      $unwind: '$product'
    },
    {
      $project: {
        productId: '$_id',
        name: '$product.name',
        sku: '$product.sku',
        totalQuantity: 1,
        totalRevenue: 1,
        salesCount: 1
      }
    }
  ]);
};
```

### 10.4 Performance Monitoring

```javascript
// Performance monitoring middleware
import { performance } from 'perf_hooks';

export const performanceMonitor = (req, res, next) => {
  const start = performance.now();

  // Capture response
  const originalSend = res.send;
  res.send = function(data) {
    const duration = performance.now() - start;

    // Log slow requests
    if (duration > 1000) {  // > 1 second
      logger.warn({
        type: 'slow_request',
        method: req.method,
        url: req.originalUrl,
        duration: `${duration.toFixed(2)}ms`,
        userId: req.userId
      });
    }

    // Send metrics to monitoring service
    metrics.timing('api.response_time', duration, {
      method: req.method,
      route: req.route?.path,
      status: res.statusCode
    });

    originalSend.call(this, data);
  };

  next();
};

// Database query performance tracking
mongoose.set('debug', (collectionName, method, query, doc, options) => {
  const start = Date.now();
  
  return function() {
    const duration = Date.now() - start;
    
    if (duration > 100) {  // > 100ms
      logger.warn({
        type: 'slow_query',
        collection: collectionName,
        method,
        duration: `${duration}ms`,
        query: JSON.stringify(query)
      });
    }
  };
});
```

### 10.5 Load Testing Results

```
Target: 1000 concurrent users, 10,000 requests/minute

Endpoint Performance (p95):
┌─────────────────────────────┬──────────┬──────────┬──────────┐
│ Endpoint                    │ p50      │ p95      │ p99      │
├─────────────────────────────┼──────────┼──────────┼──────────┤
│ GET /products               │ 45ms     │ 120ms    │ 250ms    │
│ POST /products              │ 80ms     │ 180ms    │ 350ms    │
│ GET /sales                  │ 60ms     │ 150ms    │ 300ms    │
│ POST /sales                 │ 95ms     │ 220ms    │ 450ms    │
│ GET /analytics/dashboard    │ 180ms    │ 420ms    │ 800ms    │
│ GET /analytics/forecast     │ 1200ms   │ 2500ms   │ 4000ms   │
│ GET /price-recommendation   │ 800ms    │ 1800ms   │ 3000ms   │
└─────────────────────────────┴──────────┴──────────┴──────────┘

Resource Utilization:
- API Servers: 45% CPU, 60% Memory (3 instances)
- ML Service: 70% CPU, 55% Memory (2 instances)
- Database: 35% CPU, 50% Memory
- Redis: 15% CPU, 30% Memory

Throughput:
- Sustained: 8,500 req/min
- Peak: 12,000 req/min
- Error rate: 0.02%
```

### 10.6 Performance Optimization Checklist

**Backend Optimizations:**
- [x] Database indexes on all query fields
- [x] Connection pooling (MongoDB, Redis)
- [x] Query result caching (Redis)
- [x] Pagination for large datasets
- [x] Lean queries (return plain objects)
- [x] Aggregation pipeline optimization
- [x] Async/await for I/O operations
- [x] Compression middleware (gzip)
- [x] Static asset caching
- [x] CDN for frontend assets

**Frontend Optimizations:**
- [x] Code splitting and lazy loading
- [x] React.memo for expensive components
- [x] Virtual scrolling for large lists
- [x] Debouncing for search inputs
- [x] Image optimization and lazy loading
- [x] Service worker for offline support
- [x] Bundle size optimization (<200KB)
- [x] Tree shaking unused code

**ML Service Optimizations:**
- [x] Model caching (avoid retraining)
- [x] Batch prediction requests
- [x] Async task queue for long operations
- [x] Model quantization for faster inference
- [x] Feature preprocessing caching
- [x] Parallel processing with multiprocessing


## 11. Monitoring & Observability

### 11.1 Monitoring Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                  MONITORING STACK                            │
└─────────────────────────────────────────────────────────────┘

Application Metrics → CloudWatch / Datadog
├── Request rate (req/sec)
├── Response time (p50, p95, p99)
├── Error rate (%)
├── Active users
└── API endpoint usage

Infrastructure Metrics → CloudWatch
├── CPU utilization (%)
├── Memory utilization (%)
├── Network I/O (MB/s)
├── Disk I/O (IOPS)
└── Container health

Database Metrics → MongoDB Atlas
├── Query performance
├── Connection pool usage
├── Index efficiency
├── Storage size
└── Replication lag

Application Logs → CloudWatch Logs / ELK Stack
├── Error logs
├── Access logs
├── Security events
├── Audit logs
└── Debug logs

Distributed Tracing → AWS X-Ray / Jaeger
├── Request flow
├── Service dependencies
├── Bottleneck identification
└── Error propagation

Alerting → PagerDuty / Slack
├── Critical errors
├── Performance degradation
├── Security incidents
└── Infrastructure issues
```

### 11.2 Logging Implementation

```javascript
// Structured logging with Winston
import winston from 'winston';
import { ElasticsearchTransport } from 'winston-elasticsearch';

const esTransportOpts = {
  level: 'info',
  clientOpts: {
    node: process.env.ELASTICSEARCH_URL,
    auth: {
      username: process.env.ES_USERNAME,
      password: process.env.ES_PASSWORD
    }
  },
  index: 'retail-copilot-logs'
};

export const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: {
    service: 'retail-api',
    environment: process.env.NODE_ENV
  },
  transports: [
    // Console for development
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      )
    }),
    // File for production
    new winston.transports.File({ 
      filename: 'logs/error.log', 
      level: 'error' 
    }),
    new winston.transports.File({ 
      filename: 'logs/combined.log' 
    }),
    // Elasticsearch for centralized logging
    new ElasticsearchTransport(esTransportOpts)
  ]
});

// Request logging middleware
export const requestLogger = (req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;

    logger.info({
      type: 'http_request',
      method: req.method,
      url: req.originalUrl,
      status: res.statusCode,
      duration: `${duration}ms`,
      userId: req.userId,
      ip: req.ip,
      userAgent: req.get('user-agent')
    });
  });

  next();
};

// Error logging
export const errorLogger = (err, req, res, next) => {
  logger.error({
    type: 'error',
    message: err.message,
    stack: err.stack,
    url: req.originalUrl,
    method: req.method,
    userId: req.userId,
    body: req.body
  });

  next(err);
};
```

### 11.3 Metrics Collection

```javascript
// Custom metrics with StatsD/Datadog
import StatsD from 'hot-shots';

const metrics = new StatsD({
  host: process.env.STATSD_HOST,
  port: 8125,
  prefix: 'retail_copilot.',
  globalTags: {
    env: process.env.NODE_ENV,
    service: 'api'
  }
});

// Business metrics
export const trackBusinessMetrics = {
  recordSale: (amount, userId) => {
    metrics.increment('sales.count', 1, { userId });
    metrics.histogram('sales.amount', amount, { userId });
  },

  recordForecast: (productId, confidence) => {
    metrics.increment('forecasts.generated', 1);
    metrics.gauge('forecasts.confidence', confidence, { productId });
  },

  recordPriceChange: (productId, oldPrice, newPrice) => {
    const change = ((newPrice - oldPrice) / oldPrice) * 100;
    metrics.histogram('pricing.change_pct', change, { productId });
  },

  recordUserActivity: (userId, action) => {
    metrics.increment('user.activity', 1, { userId, action });
  }
};

// System metrics
export const trackSystemMetrics = {
  recordCacheHit: (cacheType) => {
    metrics.increment('cache.hit', 1, { type: cacheType });
  },

  recordCacheMiss: (cacheType) => {
    metrics.increment('cache.miss', 1, { type: cacheType });
  },

  recordDatabaseQuery: (collection, duration) => {
    metrics.timing('database.query', duration, { collection });
  },

  recordMLPrediction: (modelType, duration) => {
    metrics.timing('ml.prediction', duration, { model: modelType });
  }
};

// Middleware to track API metrics
export const metricsMiddleware = (req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    const route = req.route?.path || 'unknown';

    metrics.timing('api.response_time', duration, {
      method: req.method,
      route,
      status: res.statusCode
    });

    metrics.increment('api.requests', 1, {
      method: req.method,
      route,
      status: res.statusCode
    });
  });

  next();
};
```

### 11.4 Health Checks

```javascript
// Comprehensive health check endpoint
export const healthCheck = async (req, res) => {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    service: 'retail-copilot-api',
    version: process.env.APP_VERSION || '1.0.0',
    checks: {}
  };

  // Database health
  try {
    await mongoose.connection.db.admin().ping();
    health.checks.database = {
      status: 'healthy',
      responseTime: '< 10ms'
    };
  } catch (error) {
    health.status = 'unhealthy';
    health.checks.database = {
      status: 'unhealthy',
      error: error.message
    };
  }

  // Redis health
  try {
    await redis.ping();
    health.checks.redis = {
      status: 'healthy',
      responseTime: '< 5ms'
    };
  } catch (error) {
    health.status = 'degraded';
    health.checks.redis = {
      status: 'unhealthy',
      error: error.message
    };
  }

  // ML service health
  try {
    const mlResponse = await axios.get(
      `${process.env.ML_SERVICE_URL}/health`,
      { timeout: 5000 }
    );
    health.checks.mlService = {
      status: 'healthy',
      version: mlResponse.data.version
    };
  } catch (error) {
    health.status = 'degraded';
    health.checks.mlService = {
      status: 'unhealthy',
      error: error.message
    };
  }

  // Memory usage
  const memUsage = process.memoryUsage();
  health.checks.memory = {
    status: memUsage.heapUsed / memUsage.heapTotal < 0.9 ? 'healthy' : 'warning',
    heapUsed: `${Math.round(memUsage.heapUsed / 1024 / 1024)}MB`,
    heapTotal: `${Math.round(memUsage.heapTotal / 1024 / 1024)}MB`
  };

  const statusCode = health.status === 'healthy' ? 200 : 503;
  res.status(statusCode).json(health);
};

// Readiness probe (for Kubernetes)
export const readinessCheck = async (req, res) => {
  try {
    // Check if service can handle requests
    await mongoose.connection.db.admin().ping();
    res.status(200).json({ ready: true });
  } catch (error) {
    res.status(503).json({ ready: false, error: error.message });
  }
};

// Liveness probe (for Kubernetes)
export const livenessCheck = (req, res) => {
  // Simple check that process is running
  res.status(200).json({ alive: true });
};
```

### 11.5 Alerting Rules

```yaml
# CloudWatch Alarms Configuration
alarms:
  # High error rate
  - name: HighErrorRate
    metric: ErrorCount
    threshold: 10
    period: 300  # 5 minutes
    evaluation_periods: 2
    statistic: Sum
    comparison: GreaterThanThreshold
    actions:
      - send_to_pagerduty
      - send_to_slack
    severity: critical

  # Slow response time
  - name: SlowResponseTime
    metric: ResponseTime
    threshold: 1000  # 1 second
    period: 300
    evaluation_periods: 3
    statistic: Average
    comparison: GreaterThanThreshold
    actions:
      - send_to_slack
    severity: warning

  # High CPU usage
  - name: HighCPUUsage
    metric: CPUUtilization
    threshold: 80
    period: 300
    evaluation_periods: 2
    statistic: Average
    comparison: GreaterThanThreshold
    actions:
      - trigger_autoscaling
      - send_to_slack
    severity: warning

  # Database connection errors
  - name: DatabaseConnectionErrors
    metric: DatabaseErrors
    threshold: 5
    period: 60
    evaluation_periods: 1
    statistic: Sum
    comparison: GreaterThanThreshold
    actions:
      - send_to_pagerduty
      - send_to_slack
    severity: critical

  # ML service unavailable
  - name: MLServiceDown
    metric: MLServiceHealthCheck
    threshold: 1
    period: 60
    evaluation_periods: 3
    statistic: Sum
    comparison: LessThanThreshold
    actions:
      - send_to_pagerduty
      - restart_ml_service
    severity: critical
```

### 11.6 Distributed Tracing

```javascript
// AWS X-Ray integration
import AWSXRay from 'aws-xray-sdk-core';
import AWS from 'aws-sdk';

// Instrument AWS SDK
const instrumentedAWS = AWSXRay.captureAWS(AWS);

// Instrument HTTP requests
import http from 'http';
import https from 'https';
AWSXRay.captureHTTPsGlobal(http);
AWSXRay.captureHTTPsGlobal(https);

// Express middleware
import xrayExpress from 'aws-xray-sdk-express';
app.use(xrayExpress.openSegment('RetailCopilot'));

// Custom subsegments for detailed tracing
export const traceAsyncOperation = async (name, operation) => {
  const segment = AWSXRay.getSegment();
  const subsegment = segment.addNewSubsegment(name);

  try {
    const result = await operation();
    subsegment.close();
    return result;
  } catch (error) {
    subsegment.addError(error);
    subsegment.close();
    throw error;
  }
};

// Usage example
const getForecast = async (productId) => {
  return await traceAsyncOperation('GetForecast', async () => {
    // Fetch sales data
    const sales = await traceAsyncOperation('FetchSalesData', async () => {
      return await Sale.find({ productId }).lean();
    });

    // Call ML service
    const forecast = await traceAsyncOperation('MLPrediction', async () => {
      return await mlService.getForecast(productId, sales);
    });

    return forecast;
  });
};

app.use(xrayExpress.closeSegment());
```

### 11.7 Dashboard Metrics

```javascript
// Real-time dashboard metrics
export const getDashboardMetrics = async () => {
  const now = new Date();
  const last24h = new Date(now - 24 * 60 * 60 * 1000);

  return {
    // System health
    system: {
      status: 'healthy',
      uptime: process.uptime(),
      activeConnections: mongoose.connection.readyState,
      memoryUsage: process.memoryUsage().heapUsed / 1024 / 1024
    },

    // API metrics (last 24h)
    api: {
      totalRequests: await getMetricValue('api.requests', last24h),
      avgResponseTime: await getMetricValue('api.response_time.avg', last24h),
      errorRate: await getMetricValue('api.error_rate', last24h),
      requestsPerMinute: await getMetricValue('api.rpm', last24h)
    },

    // Business metrics (last 24h)
    business: {
      totalSales: await Sale.countDocuments({ 
        saleDate: { $gte: last24h } 
      }),
      totalRevenue: await Sale.aggregate([
        { $match: { saleDate: { $gte: last24h } } },
        { $group: { _id: null, total: { $sum: '$totalAmount' } } }
      ]),
      activeUsers: await getActiveUserCount(last24h),
      forecastsGenerated: await Forecast.countDocuments({
        createdAt: { $gte: last24h }
      })
    },

    // ML metrics
    ml: {
      avgForecastTime: await getMetricValue('ml.prediction.avg', last24h),
      avgConfidence: await Forecast.aggregate([
        { $match: { createdAt: { $gte: last24h } } },
        { $group: { _id: null, avg: { $avg: '$confidence' } } }
      ]),
      modelsInProduction: await getActiveModelCount()
    },

    // Database metrics
    database: {
      totalDocuments: await getTotalDocumentCount(),
      avgQueryTime: await getMetricValue('database.query.avg', last24h),
      connectionPoolUsage: mongoose.connection.client.topology.s.pool.totalConnectionCount
    }
  };
};
```


## 12. Disaster Recovery

### 12.1 Backup Strategy

```
┌─────────────────────────────────────────────────────────────┐
│                    BACKUP ARCHITECTURE                       │
└─────────────────────────────────────────────────────────────┘

Database Backups (MongoDB Atlas):
├── Continuous Backup (Point-in-Time Recovery)
│   ├── Frequency: Every 6 hours
│   ├── Retention: 7 days
│   └── Recovery: Any point within retention period
├── Snapshot Backups
│   ├── Frequency: Daily at 2 AM UTC
│   ├── Retention: 30 days
│   └── Storage: AWS S3 (encrypted)
└── Weekly Full Backups
    ├── Frequency: Sunday 2 AM UTC
    ├── Retention: 90 days
    └── Storage: AWS S3 Glacier (long-term)

Application Data Backups:
├── ML Models
│   ├── Frequency: After each training
│   ├── Retention: Last 5 versions
│   └── Storage: S3 with versioning
├── User Uploads (CSV, Reports)
│   ├── Frequency: Real-time replication
│   ├── Retention: Indefinite
│   └── Storage: S3 with versioning
└── Configuration Files
    ├── Frequency: On change (Git)
    ├── Retention: Full history
    └── Storage: GitHub repository

Infrastructure as Code:
├── Terraform State
│   ├── Storage: S3 with versioning
│   ├── Locking: DynamoDB
│   └── Encryption: AES-256
└── Kubernetes Manifests
    ├── Storage: Git repository
    └── Backup: GitHub + GitLab mirror
```

### 12.2 Recovery Procedures

#### Database Recovery

```bash
#!/bin/bash
# Database recovery script

# Restore from snapshot
restore_from_snapshot() {
  SNAPSHOT_ID=$1
  TARGET_CLUSTER=$2
  
  echo "Restoring MongoDB from snapshot: $SNAPSHOT_ID"
  
  # Using MongoDB Atlas API
  curl -X POST \
    "https://cloud.mongodb.com/api/atlas/v1.0/groups/${PROJECT_ID}/clusters/${TARGET_CLUSTER}/backup/restoreJobs" \
    -u "${ATLAS_PUBLIC_KEY}:${ATLAS_PRIVATE_KEY}" \
    -H "Content-Type: application/json" \
    -d "{
      \"snapshotId\": \"${SNAPSHOT_ID}\",
      \"deliveryType\": \"automated\",
      \"targetClusterName\": \"${TARGET_CLUSTER}\"
    }"
  
  echo "Restore job initiated. Monitor progress in Atlas console."
}

# Point-in-time recovery
restore_point_in_time() {
  TIMESTAMP=$1
  TARGET_CLUSTER=$2
  
  echo "Restoring MongoDB to point-in-time: $TIMESTAMP"
  
  curl -X POST \
    "https://cloud.mongodb.com/api/atlas/v1.0/groups/${PROJECT_ID}/clusters/${TARGET_CLUSTER}/backup/restoreJobs" \
    -u "${ATLAS_PUBLIC_KEY}:${ATLAS_PRIVATE_KEY}" \
    -H "Content-Type: application/json" \
    -d "{
      \"pointInTimeUTCSeconds\": ${TIMESTAMP},
      \"deliveryType\": \"automated\",
      \"targetClusterName\": \"${TARGET_CLUSTER}\"
    }"
}

# Verify restoration
verify_restoration() {
  TARGET_CLUSTER=$1
  
  echo "Verifying database restoration..."
  
  # Check document counts
  mongo "${MONGODB_URI}" --eval "
    db.users.countDocuments();
    db.products.countDocuments();
    db.sales.countDocuments();
  "
  
  # Run data integrity checks
  node scripts/verify-data-integrity.js
}
```

#### Application Recovery

```bash
#!/bin/bash
# Application recovery script

# Rollback to previous version
rollback_deployment() {
  SERVICE=$1
  PREVIOUS_VERSION=$2
  
  echo "Rolling back $SERVICE to version $PREVIOUS_VERSION"
  
  # Update ECS service
  aws ecs update-service \
    --cluster retail-cluster \
    --service $SERVICE \
    --task-definition ${SERVICE}:${PREVIOUS_VERSION} \
    --force-new-deployment
  
  # Wait for deployment to complete
  aws ecs wait services-stable \
    --cluster retail-cluster \
    --services $SERVICE
  
  echo "Rollback completed"
}

# Restore from backup
restore_application_data() {
  BACKUP_DATE=$1
  
  echo "Restoring application data from $BACKUP_DATE"
  
  # Restore ML models
  aws s3 sync \
    s3://retail-backups/ml-models/${BACKUP_DATE}/ \
    /app/models/ \
    --delete
  
  # Restore configuration
  aws s3 cp \
    s3://retail-backups/config/${BACKUP_DATE}/config.json \
    /app/config/
  
  echo "Application data restored"
}

# Full system recovery
full_system_recovery() {
  RECOVERY_POINT=$1
  
  echo "Initiating full system recovery to $RECOVERY_POINT"
  
  # 1. Restore database
  restore_from_snapshot $RECOVERY_POINT retail-cluster-recovery
  
  # 2. Restore application data
  restore_application_data $RECOVERY_POINT
  
  # 3. Deploy services
  kubectl apply -f k8s/production/
  
  # 4. Verify health
  ./scripts/health-check.sh
  
  echo "Full system recovery completed"
}
```

### 12.3 Disaster Recovery Plan

#### Recovery Time Objective (RTO) & Recovery Point Objective (RPO)

| Component | RTO | RPO | Recovery Strategy |
|-----------|-----|-----|-------------------|
| Database | 1 hour | 6 hours | Automated snapshot restore |
| API Service | 15 minutes | 0 (stateless) | Redeploy from container registry |
| ML Service | 30 minutes | 0 (stateless) | Redeploy + restore models |
| Frontend | 5 minutes | 0 | Redeploy from S3/CDN |
| User Data | 1 hour | 6 hours | Database restore |
| ML Models | 2 hours | 24 hours | S3 restore + retrain if needed |

#### Disaster Scenarios & Response

**Scenario 1: Database Failure**
```
Detection: Health check fails, connection errors
Impact: Complete service outage
Response:
1. Activate standby database replica (automatic failover)
2. If replica unavailable, restore from latest snapshot
3. Verify data integrity
4. Resume service
Timeline: 15-60 minutes
```

**Scenario 2: Region Outage (AWS)**
```
Detection: Multiple service failures, AWS status page
Impact: Complete service outage in primary region
Response:
1. Activate disaster recovery region (manual)
2. Update DNS to point to DR region
3. Restore database from cross-region backup
4. Deploy services in DR region
5. Verify functionality
Timeline: 2-4 hours
```

**Scenario 3: Data Corruption**
```
Detection: Data validation errors, user reports
Impact: Incorrect data, potential business logic errors
Response:
1. Identify corruption scope and timeline
2. Restore database to point before corruption
3. Replay valid transactions from logs
4. Verify data integrity
5. Notify affected users
Timeline: 2-6 hours
```

**Scenario 4: Security Breach**
```
Detection: Security alerts, unusual activity
Impact: Potential data exposure, service compromise
Response:
1. Isolate affected systems immediately
2. Revoke all access tokens
3. Rotate all secrets and credentials
4. Audit access logs
5. Restore from clean backup if needed
6. Notify users and authorities as required
Timeline: 1-24 hours (depending on severity)
```

### 12.4 Failover Architecture

```
┌─────────────────────────────────────────────────────────────┐
│              MULTI-REGION FAILOVER                           │
└─────────────────────────────────────────────────────────────┘

Primary Region (us-east-1):
├── Active services
├── Primary database (read/write)
├── Real-time replication to DR
└── Health monitoring

Disaster Recovery Region (us-west-2):
├── Standby services (scaled to 0)
├── Read replica database
├── Receives continuous replication
└── Ready for activation

Failover Process:
1. Health check detects primary region failure
2. Route 53 health check fails (3 consecutive)
3. DNS automatically updates to DR region
4. DR services auto-scale to production capacity
5. Database replica promoted to primary
6. Services resume in DR region

Failback Process:
1. Primary region restored and verified
2. Sync data from DR to primary
3. Gradually shift traffic back (10% increments)
4. Monitor for issues
5. Complete failback when stable
6. DR region returns to standby mode
```

### 12.5 Testing & Validation

```javascript
// Disaster recovery testing script
class DisasterRecoveryTest {
  async runTests() {
    console.log('Starting DR tests...');

    // Test 1: Database backup and restore
    await this.testDatabaseRestore();

    // Test 2: Service failover
    await this.testServiceFailover();

    // Test 3: Data integrity after restore
    await this.testDataIntegrity();

    // Test 4: Recovery time measurement
    await this.measureRecoveryTime();

    console.log('DR tests completed');
  }

  async testDatabaseRestore() {
    console.log('Testing database restore...');

    // Create test data
    const testData = await this.createTestData();

    // Trigger backup
    await this.triggerBackup();

    // Simulate data loss
    await this.simulateDataLoss();

    // Restore from backup
    const restoreStart = Date.now();
    await this.restoreFromBackup();
    const restoreTime = Date.now() - restoreStart;

    // Verify data
    const dataIntact = await this.verifyTestData(testData);

    console.log(`Restore completed in ${restoreTime}ms`);
    console.log(`Data integrity: ${dataIntact ? 'PASS' : 'FAIL'}`);
  }

  async testServiceFailover() {
    console.log('Testing service failover...');

    // Simulate primary region failure
    await this.simulateRegionFailure('us-east-1');

    // Wait for failover
    await this.waitForFailover();

    // Verify services in DR region
    const servicesHealthy = await this.checkServicesHealth('us-west-2');

    console.log(`Failover: ${servicesHealthy ? 'PASS' : 'FAIL'}`);
  }

  async measureRecoveryTime() {
    const scenarios = [
      { name: 'Database Restore', target: 3600000 },  // 1 hour
      { name: 'Service Failover', target: 900000 },   // 15 minutes
      { name: 'Full Recovery', target: 7200000 }      // 2 hours
    ];

    for (const scenario of scenarios) {
      const actualTime = await this.runRecoveryScenario(scenario.name);
      const status = actualTime <= scenario.target ? 'PASS' : 'FAIL';
      
      console.log(`${scenario.name}: ${actualTime}ms (${status})`);
    }
  }
}

// Schedule quarterly DR tests
cron.schedule('0 2 1 */3 *', async () => {
  const drTest = new DisasterRecoveryTest();
  await drTest.runTests();
});
```

### 12.6 Incident Response Runbook

```markdown
# Incident Response Runbook

## Severity Levels

**P0 - Critical**: Complete service outage, data loss
**P1 - High**: Major functionality impaired, affecting >50% users
**P2 - Medium**: Partial functionality impaired, affecting <50% users
**P3 - Low**: Minor issues, workarounds available

## Response Procedures

### P0 - Critical Incident

1. **Immediate Actions** (0-5 minutes)
   - [ ] Declare incident in Slack #incidents channel
   - [ ] Page on-call engineer via PagerDuty
   - [ ] Create incident ticket
   - [ ] Start incident bridge call

2. **Assessment** (5-15 minutes)
   - [ ] Identify affected systems
   - [ ] Determine root cause (if obvious)
   - [ ] Estimate impact (users, data, revenue)
   - [ ] Decide on recovery strategy

3. **Communication** (15-30 minutes)
   - [ ] Update status page
   - [ ] Notify customers via email
   - [ ] Post updates every 30 minutes
   - [ ] Escalate to leadership

4. **Recovery** (30 minutes - 4 hours)
   - [ ] Execute recovery plan
   - [ ] Monitor recovery progress
   - [ ] Verify service restoration
   - [ ] Confirm data integrity

5. **Post-Incident** (24-48 hours)
   - [ ] Write post-mortem
   - [ ] Identify action items
   - [ ] Update runbooks
   - [ ] Schedule follow-up review

## Contact Information

- On-Call Engineer: PagerDuty rotation
- Engineering Manager: [phone]
- CTO: [phone]
- AWS Support: [account number]
- MongoDB Support: [account number]
```

---

## Appendix

### A. Technology Versions

| Technology | Version | Release Date | EOL Date |
|------------|---------|--------------|----------|
| Node.js | 18 LTS | 2022-10-25 | 2025-04-30 |
| React | 18.2 | 2022-06-14 | N/A |
| MongoDB | 5.0 | 2021-07-13 | 2024-10-31 |
| Python | 3.9 | 2020-10-05 | 2025-10-05 |
| Redis | 7.0 | 2022-04-27 | N/A |

### B. Glossary

- **RTO**: Recovery Time Objective - Maximum acceptable downtime
- **RPO**: Recovery Point Objective - Maximum acceptable data loss
- **MTTR**: Mean Time To Recovery - Average time to restore service
- **MTBF**: Mean Time Between Failures - Average time between incidents
- **SLA**: Service Level Agreement - Guaranteed uptime percentage
- **SLO**: Service Level Objective - Target performance metrics
- **SLI**: Service Level Indicator - Measured performance metrics

### C. References

1. MongoDB Performance Best Practices - https://docs.mongodb.com/manual/administration/analyzing-mongodb-performance/
2. Node.js Production Best Practices - https://nodejs.org/en/docs/guides/nodejs-docker-webapp/
3. AWS Well-Architected Framework - https://aws.amazon.com/architecture/well-architected/
4. React Performance Optimization - https://react.dev/learn/render-and-commit
5. scikit-learn Best Practices - https://scikit-learn.org/stable/developers/



