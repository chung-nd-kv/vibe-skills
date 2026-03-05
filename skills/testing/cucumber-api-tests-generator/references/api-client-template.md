# API Client Template

## BaseApiClient

Shared base class mọi domain client đều extend. Handles auth headers, base URL, error wrapping.

```typescript
// src/api_clients/base_api_client.ts
import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse } from 'axios';
import config from '@config/index';

export interface ApiResponse<T = unknown> {
  status: number;
  data: T;
  headers: Record<string, string>;
  ok: boolean;           // status >= 200 && status < 300
  error?: string;        // error message if request failed
}

export class BaseApiClient {
  protected client: AxiosInstance;
  private authToken?: string;

  constructor(baseURL?: string) {
    this.client = axios.create({
      baseURL: baseURL ?? config.api.baseUrl,
      timeout: config.api.timeout ?? 30000,
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
      },
      // IMPORTANT: Don't throw on non-2xx — return status in ApiResponse
      validateStatus: () => true,
    });
  }

  setAuthToken(token: string): void {
    this.authToken = token;
  }

  clearAuthToken(): void {
    this.authToken = undefined;
  }

  protected buildHeaders(extraHeaders?: Record<string, string>): Record<string, string> {
    const headers: Record<string, string> = { ...extraHeaders };
    if (this.authToken) {
      headers['Authorization'] = `Bearer ${this.authToken}`;
    }
    return headers;
  }

  protected async request<T>(
    method: string,
    endpoint: string,
    options?: {
      body?: unknown;
      queryParams?: Record<string, string | number>;
      headers?: Record<string, string>;
    }
  ): Promise<ApiResponse<T>> {
    const requestConfig: AxiosRequestConfig = {
      method: method.toLowerCase(),
      url: endpoint,
      headers: this.buildHeaders(options?.headers),
      params: options?.queryParams,
      data: options?.body,
    };

    try {
      const response: AxiosResponse<T> = await this.client.request(requestConfig);
      return {
        status: response.status,
        data: response.data,
        headers: response.headers as Record<string, string>,
        ok: response.status >= 200 && response.status < 300,
      };
    } catch (error) {
      // Network errors, timeouts — not HTTP errors
      const message = error instanceof Error ? error.message : 'Unknown error';
      return {
        status: 0,
        data: null as T,
        headers: {},
        ok: false,
        error: `Network error: ${message}`,
      };
    }
  }

  // Convenience methods
  protected get<T>(endpoint: string, queryParams?: Record<string, string | number>, headers?: Record<string, string>) {
    return this.request<T>('GET', endpoint, { queryParams, headers });
  }

  protected post<T>(endpoint: string, body?: unknown, headers?: Record<string, string>) {
    return this.request<T>('POST', endpoint, { body, headers });
  }

  protected put<T>(endpoint: string, body?: unknown, headers?: Record<string, string>) {
    return this.request<T>('PUT', endpoint, { body, headers });
  }

  protected patch<T>(endpoint: string, body?: unknown, headers?: Record<string, string>) {
    return this.request<T>('PATCH', endpoint, { body, headers });
  }

  protected delete<T>(endpoint: string, headers?: Record<string, string>) {
    return this.request<T>('DELETE', endpoint, { headers });
  }
}
```

---

## Domain API Client Example: ProductApiClient

```typescript
// src/api_clients/product_api_client.ts
import { BaseApiClient, ApiResponse } from './base_api_client';
import {
  CreateProductRequest,
  UpdateProductRequest,
  ProductResponse,
  ProductListResponse,
} from '@models/api/product';

export class ProductApiClient extends BaseApiClient {
  private readonly basePath = '/api/v1/products';

  async getProducts(params?: {
    page?: number;
    pageSize?: number;
    keyword?: string;
  }): Promise<ApiResponse<ProductListResponse>> {
    return this.get<ProductListResponse>(this.basePath, params as Record<string, string | number>);
  }

  async getProductById(id: string): Promise<ApiResponse<ProductResponse>> {
    return this.get<ProductResponse>(`${this.basePath}/${id}`);
  }

  async createProduct(request: CreateProductRequest): Promise<ApiResponse<ProductResponse>> {
    return this.post<ProductResponse>(this.basePath, request);
  }

  async updateProduct(id: string, request: UpdateProductRequest): Promise<ApiResponse<ProductResponse>> {
    return this.put<ProductResponse>(`${this.basePath}/${id}`, request);
  }

  async deleteProduct(id: string): Promise<ApiResponse<void>> {
    return this.delete<void>(`${this.basePath}/${id}`);
  }
}
```

---

## Auth API Client

```typescript
// src/api_clients/auth_api_client.ts
import { BaseApiClient, ApiResponse } from './base_api_client';

interface LoginRequest {
  username: string;
  password: string;
}

interface LoginResponse {
  accessToken: string;
  refreshToken: string;
  expiresIn: number;
}

export class AuthApiClient extends BaseApiClient {
  async login(credentials: LoginRequest): Promise<ApiResponse<LoginResponse>> {
    return this.post<LoginResponse>('/api/v1/auth/login', credentials);
  }

  async refreshToken(refreshToken: string): Promise<ApiResponse<LoginResponse>> {
    return this.post<LoginResponse>('/api/v1/auth/refresh', { refreshToken });
  }
}

// Singleton for use in steps
export const authApiClient = new AuthApiClient();
```

---

## World Context Extension

Add these fields to `CustomWorld` for API test state:

```typescript
// src/support/world.ts (additions)
import { ApiResponse } from '@api_clients/base_api_client';
import { BaseApiClient } from '@api_clients/base_api_client';

export class CustomWorld extends World {
  // ... existing fields ...

  // API test state
  authToken?: string;
  apiResponse?: ApiResponse<unknown>;
  lastRequestBody?: unknown;
  lastEndpoint?: string;

  // Dynamic client registry
  private apiClients: Map<string, BaseApiClient> = new Map();

  getApiClient<T extends BaseApiClient>(ClientClass: new () => T): T {
    const key = ClientClass.name;
    if (!this.apiClients.has(key)) {
      const client = new ClientClass();
      if (this.authToken) client.setAuthToken(this.authToken);
      this.apiClients.set(key, client);
    }
    return this.apiClients.get(key) as T;
  }

  // Re-inject auth token into all active clients
  refreshAllClientTokens(): void {
    if (this.authToken) {
      this.apiClients.forEach(client => client.setAuthToken(this.authToken!));
    }
  }
}
```

---

## Config Setup

```typescript
// src/config/index.ts (additions for API)
export default {
  api: {
    baseUrl: process.env.API_BASE_URL ?? 'http://localhost:5000',
    timeout: Number(process.env.API_TIMEOUT) || 30000,
  },
  // ... existing config ...
};
```

---

## Dependencies

```json
{
  "dependencies": {
    "axios": "^1.6.0"
  },
  "devDependencies": {
    "@types/axios": "^0.14.0"
  }
}
```
