# 🧪 MEGA PROJECT 03 (OPTIONAL)
# **E-SHOP MICROSERVICES**

## 📖 1. Giới thiệu Microservices

**Microservices** là kiến trúc chia nhỏ ứng dụng lớn thành các dịch vụ nhỏ, độc lập, giao tiếp với nhau qua mạng.

### Khi nào dùng Microservices?
- Hệ thống quá lớn, team quá đông.
- Cần scale từng phần riêng biệt (ví dụ: module Search cần nhiều CPU hơn module Login).
- Công nghệ đa dạng (Service A dùng .NET, Service B dùng Node.js).

> [!CAUTION]
> Microservices rất phức tạp! Chỉ nên học sau khi đã thành thạo Monolith. Đừng dùng dao mổ trâu để giết gà.

---

## 🏗️ 2. Kiến trúc hệ thống

Chúng ta sẽ tách E-Shop thành các services sau:

1. **Catalog Service**: Quản lý sản phẩm, danh mục. (SQL Server)
2. **Basket Service**: Quản lý giỏ hàng tạm thời. (Redis)
3. **Identity Service**: Quản lý User, Login, cấp Token. (SQL Server)
4. **Ordering Service**: Xử lý đặt hàng. (SQL Server)
5. **API Gateway**: Cổng vào duy nhất cho Client. (Ocelot)

---

## 🛠️ 3. Công nghệ sử dụng

- **ASP.NET Core Web API**: Nền tảng chính.
- **Docker**: Đóng gói từng service thành container.
- **Docker Compose**: Chạy toàn bộ hệ thống bằng 1 lệnh.
- **Ocelot**: API Gateway.
- **RabbitMQ** (Nâng cao): Giao tiếp bất đồng bộ giữa các service.

---

## 🚀 4. Hướng dẫn thực hiện

### Bước 1: Tạo Solution và Folders
```
EShop.Microservices/
├── Services/
│   ├── Catalog/
│   ├── Basket/
│   ├── Identity/
│   └── Ordering/
├── Gateways/
│   └── ApiGateway/
└── docker-compose.yml
```

### Bước 2: Xây dựng Catalog Service
- Là một Web API độc lập.
- Có DbContext riêng, Database riêng.
- Chỉ có API CRUD Product.
- Chạy trên port 5001.

### Bước 3: Xây dựng Basket Service
- Sử dụng **Redis** để lưu giỏ hàng (Key-Value store) thay vì SQL.
- Tốc độ cực nhanh.
- Chạy trên port 5002.

### Bước 4: Cấu hình API Gateway (Ocelot)
Tạo project `ApiGateway` trống, cài `Ocelot`.
Cấu hình `ocelot.json`:

```json
{
  "Routes": [
    {
      "DownstreamPathTemplate": "/api/products",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [ { "Host": "catalog-service", "Port": 80 } ],
      "UpstreamPathTemplate": "/api/products",
      "UpstreamHttpMethod": [ "Get" ]
    }
  ]
}
```
Client chỉ cần gọi vào Gateway, Gateway sẽ tự điều hướng sang Catalog Service.

### Bước 5: Dockerize (Container hóa)
Tạo `Dockerfile` cho từng Service:

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["Services/Catalog/Catalog.API.csproj", "Services/Catalog/"]
RUN dotnet restore "Services/Catalog/Catalog.API.csproj"
COPY . .
RUN dotnet build "Services/Catalog/Catalog.API.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "Services/Catalog/Catalog.API.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Catalog.API.dll"]
```

### Bước 6: Docker Compose
File `docker-compose.yml` để chạy tất cả:

```yaml
version: '3.4'

services:
  catalog-db:
    image: mcr.microsoft.com/mssql/server:2019-latest
    
  catalog-api:
    image: eshop/catalog-api
    build:
      context: .
      dockerfile: Services/Catalog/Dockerfile
    depends_on:
      - catalog-db
      
  basket-redis:
    image: redis:alpine
    
  basket-api:
    image: eshop/basket-api
    depends_on:
      - basket-redis
```

---

## 🧪 5. Chạy thử nghiệm

1. Cài đặt Docker Desktop.
2. Mở terminal tại thư mục gốc.
3. Chạy lệnh: `docker-compose up --build`.
4. Chờ vài phút để tải image và build.
5. Truy cập API Gateway (ví dụ `localhost:8000/api/products`).

---

## 💡 Mẹo nhỏ
> [!TIP]
> Trong môi trường Dev, bạn có thể chạy nhiều Project cùng lúc trong Visual Studio bằng cách chuột phải vào Solution -> **Set Startup Projects** -> **Multiple startup projects**.

> [!IMPORTANT]
> Giao tiếp giữa các service nên hạn chế gọi trực tiếp (HTTP Client) để tránh bị treo dây chuyền. Hãy tìm hiểu về **Event-Driven Architecture** với RabbitMQ.
