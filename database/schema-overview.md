## 📚 QuickStore Database Tables
- [users](#users-系统用户表)
- [products](#products-商品信息表)
- [customers](#customers-客户信息表)
- [inbound_shipments](#inbound_shipments-入库记录表)
- [orders](#orders-订单表)
- [order_items](#order_items-订单明细表)
- [delivery](#delivery-配送记录表)

# QuickStore Database Schema Overview

This document outlines the structure and meaning of all database tables used in the QuickStore WMS.

### **users** （系统用户）
| 字段名             | 类型          | 描述                               |
| --------------- | ----------- | -------------------------------- |
| `id`            | SERIAL (PK) | 用户 ID，自增主键                       |
| `username`      | TEXT        | 登录名（唯一）                          |
| `password_hash` | TEXT        | 加密后的密码                           |
| `full_name`     | TEXT        | 用户姓名                             |
| `role`          | TEXT        | 角色：如 'admin'、'staff'、'warehouse' |
| `created_at`    | TIMESTAMP   | 创建时间                             |

### **products** （商品信息）
| 字段名              | 类型             | 描述                  |
| ---------------- | -------------- | ------------------- |
| `id`             | SERIAL (PK)    | 产品 ID，自增主键          |
| `name`           | TEXT           | 产品名称（如 角铝、圆管 等）     |
| `specification`  | TEXT           | 规格（如 20x20mm、2mm 厚） |
| `unit`           | TEXT           | 单位（如 米、条、根）         |
| `price`          | NUMERIC(10, 2) | 单价（如 12.50）         |
| `stock_quantity` | INTEGER        | 当前库存数量              |
| `created_at`     | TIMESTAMP      | 添加时间                |

### **inbound_shipments** （入库记录）
| 字段名             | 类型                         | 描述           |
| --------------- | -------------------------- | ------------ |
| `id`            | SERIAL (PK)                | 入库记录 ID      |
| `product_id`    | INTEGER (FK → products.id) | 入库的产品        |
| `quantity`      | INTEGER                    | 入库数量         |
| `shipment_code` | TEXT                       | 货柜编号或批次号（可选） |
| `arrival_date`  | DATE                       | 到货日期         |
| `created_by`    | INTEGER (FK → users.id)    | 哪个用户录入的      |
| `created_at`    | TIMESTAMP                  | 入库登记时间       |

### **customers** （客户信息）
| 字段名              | 类型          | 描述            |
| ---------------- | ----------- | ------------- |
| `id`             | SERIAL (PK) | 客户 ID         |
| `name`           | TEXT        | 客户名称（公司名或个人名） |
| `contact_number` | TEXT        | 电话号码          |
| `email`          | TEXT        | 邮箱（可选）        |
| `address`        | TEXT        | 地址            |
| `created_at`     | TIMESTAMP   | 添加时间          |

### **orders** （客户下的订单）
| 字段名           | 类型                          | 描述                                 |
| ------------- | --------------------------- | ---------------------------------- |
| `id`          | SERIAL (PK)                 | 订单 ID                              |
| `customer_id` | INTEGER (FK → customers.id) | 所属客户                               |
| `order_date`  | TIMESTAMP                   | 下单时间                               |
| `created_by`  | INTEGER (FK → users.id)     | 哪位员工录入的                            |
| `is_delivery` | BOOLEAN                     | 是否需要配送                             |
| `status`      | TEXT                        | 状态（如：pending, fulfilled, canceled） |

### **order_items** （订单中的商品明细）
| 字段名          | 类型                         | 描述                 |
| ------------ | -------------------------- | ------------------ |
| `id`         | SERIAL (PK)                | 明细 ID              |
| `order_id`   | INTEGER (FK → orders.id)   | 所属订单               |
| `product_id` | INTEGER (FK → products.id) | 产品                 |
| `quantity`   | INTEGER                    | 数量                 |
| `unit_price` | NUMERIC(10, 2)             | 下单时的单价（防止价格变动影响历史） |

### **delivery** （配送记录，可选）
| 字段名             | 类型                       | 描述       |
| --------------- | ------------------------ | -------- |
| `id`            | SERIAL (PK)              | 配送记录 ID  |
| `order_id`      | INTEGER (FK → orders.id) | 对应的订单    |
| `delivery_date` | DATE                     | 配送日期     |
| `delivered_by`  | INTEGER (FK → users.id)  | 配送人员     |
| `is_delivered`  | BOOLEAN                  | 是否已送达    |
| `notes`         | TEXT                     | 配送备注（可选） |

各表之间的一对多关系：
| 主表                                                  | 子表        | 关系类型 |
| --------------------------------------------------- | --------- | ---- |
| `customers` → `orders`                              | 1 → ∞     |      |
| `orders` → `order_items`                            | 1 → ∞     |      |
| `products` → `order_items`                          | 1 → ∞     |      |
| `products` → `inbound_shipments`                    | 1 → ∞     |      |
| `users` → `orders`, `inbound_shipments`, `delivery` | 1 → ∞     |      |
| `orders` → `delivery`                               | 1 → 1（可选） |      |


-- 数据库表的建表语句
-- 1. 用户表
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  full_name TEXT,
  role TEXT CHECK (role IN ('admin', 'staff', 'warehouse')),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. 产品表
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  specification TEXT,
  unit TEXT,
  price NUMERIC(10, 2),
  stock_quantity INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 3. 客户表
CREATE TABLE customers (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  contact_number TEXT,
  email TEXT,
  address TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 4. 入库记录表
CREATE TABLE inbound_shipments (
  id SERIAL PRIMARY KEY,
  product_id INTEGER REFERENCES products(id),
  quantity INTEGER NOT NULL,
  shipment_code TEXT,
  arrival_date DATE,
  created_by INTEGER REFERENCES users(id),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 5. 订单表
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  customer_id INTEGER REFERENCES customers(id),
  order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_by INTEGER REFERENCES users(id),
  is_delivery BOOLEAN DEFAULT FALSE,
  status TEXT CHECK (status IN ('pending', 'fulfilled', 'canceled')) DEFAULT 'pending'
);

-- 6. 订单明细表
CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INTEGER REFERENCES orders(id),
  product_id INTEGER REFERENCES products(id),
  quantity INTEGER NOT NULL,
  unit_price NUMERIC(10, 2) NOT NULL
);

-- 7. 配送记录表（可选）
CREATE TABLE delivery (
  id SERIAL PRIMARY KEY,
  order_id INTEGER UNIQUE REFERENCES orders(id),
  delivery_date DATE,
  delivered_by INTEGER REFERENCES users(id),
  is_delivered BOOLEAN DEFAULT FALSE,
  notes TEXT
);
