# ERP 시스템 API 명세서

> 건설 현장 자재 관리 ERP 시스템 API 문서  
> Base URL: `http://localhost:3000`  
> Swagger UI: `http://localhost:3000/api-docs`

## 목차

1. [인증 (Authentication)](#1-인증-authentication)
2. [사용자 (Users)](#2-사용자-users)
3. [자재 관리 (Materials)](#3-자재-관리-materials)
4. [자재 카테고리 (Material Categories)](#4-자재-카테고리-material-categories)
5. [BOM 관리 (BOM Masters)](#5-bom-관리-bom-masters)
6. [현장 관리 (Sites)](#6-현장-관리-sites)
7. [창고 관리 (Warehouses)](#7-창고-관리-warehouses)
8. [공급사 관리 (Suppliers)](#8-공급사-관리-suppliers)
9. [재고 관리 (Stocks)](#9-재고-관리-stocks)
10. [재고 이동 (Stock Movements)](#10-재고-이동-stock-movements)
11. [재고 조정 (Stock Adjustments)](#11-재고-조정-stock-adjustments)
12. [견적 관리 (Quotations)](#12-견적-관리-quotations)
13. [발주 관리 (Purchase Orders)](#13-발주-관리-purchase-orders)
14. [시리얼 번호 관리 (Material Serials)](#14-시리얼-번호-관리-material-serials)
15. [대시보드 (Dashboard)](#15-대시보드-dashboard)
16. [리포트 (Reports)](#16-리포트-reports)

---

## 공통 사항

### 인증 방식
- JWT Bearer Token 인증
- 로그인 후 받은 `access_token`을 요청 헤더에 포함
```
Authorization: Bearer {access_token}
```

### 응답 형식
모든 API는 JSON 형식으로 응답합니다.

### 에러 응답
```json
{
  "statusCode": 400,
  "message": "에러 메시지",
  "error": "Bad Request"
}
```

### 페이징 파라미터
목록 조회 API는 다음 쿼리 파라미터를 지원합니다:
- `page`: 페이지 번호 (기본값: 1)
- `limit`: 페이지당 항목 수 (기본값: 10)

---

## 1. 인증 (Authentication)

### 1.1 회원가입
```
POST /auth/register
```

**Request Body:**
```json
{
  "userId": "user123",
  "userPwd": "password123!",
  "userName": "홍길동",
  "userEmail": "user@example.com",
  "userPhone": "010-1234-5678"
}
```

**Response (201):**
```json
{
  "id": 1,
  "userId": "user123",
  "userName": "홍길동",
  "userEmail": "user@example.com",
  "userPhone": "010-1234-5678",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

**Error (401):** 이미 존재하는 사용자

---

### 1.2 로그인
```
POST /auth/login
```

**Request Body:**
```json
{
  "userId": "user123",
  "userPwd": "password123!"
}
```

**Response (200):**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Error (401):** 인증 실패

---

### 1.3 로그아웃
```
POST /auth/logout
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "message": "로그아웃 성공"
}
```

---

### 1.4 토큰 갱신
```
POST /auth/refresh
```

**Request Body:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response (200):**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Error (401):** 유효하지 않은 토큰

---

### 1.5 비밀번호 재설정
```
POST /auth/reset-password
```

**Request Body:**
```json
{
  "userId": "user123",
  "newPassword": "newPassword123!"
}
```

**Response (200):**
```json
{
  "message": "비밀번호 재설정 성공"
}
```

**Error (404):** 사용자를 찾을 수 없음

---

## 2. 사용자 (Users)

### 2.1 사용자 정보 수정
```
PUT /users/profile
```
🔒 **인증 필요**  
📎 **multipart/form-data**

**Request Body (Form Data):**
- `userName`: string (선택)
- `userEmail`: string (선택)
- `userPhone`: string (선택)
- `image`: file (선택, JPEG/PNG/GIF, 최대 5MB)

**Response (200):**
```json
{
  "id": 1,
  "userId": "user123",
  "userName": "홍길동",
  "userEmail": "user@example.com",
  "userPhone": "010-1234-5678",
  "userImage": "/uploads/users/1234567890.jpg",
  "updatedAt": "2026-02-20T08:00:00.000Z"
}
```

---

## 3. 자재 관리 (Materials)

### 3.1 자재 목록 조회
```
GET /materials
```
🔒 **인증 필요**

**Query Parameters:**
- `page`: number (선택, 기본값: 1)
- `limit`: number (선택, 기본값: 10)
- `search`: string (선택, 자재명/코드 검색)
- `categoryId`: number (선택, 카테고리 필터)
- `manufacturer`: string (선택, 제조사 필터)
- `isActive`: boolean (선택, 활성 상태 필터)

**Response (200):**
```json
{
  "data": [
    {
      "id": 1,
      "materialCode": "MAT001",
      "materialName": "IP 카메라",
      "categoryId": 1,
      "category": {
        "id": 1,
        "categoryName": "보안장비"
      },
      "manufacturer": "Hikvision",
      "modelNumber": "DS-2CD2143G0-I",
      "specifications": {
        "resolution": "4MP",
        "lens": "2.8mm"
      },
      "unit": "EA",
      "unitPrice": 150000,
      "safetyStockLevel": 10,
      "reorderPoint": 5,
      "requiresSerial": false,
      "isBomProduct": false,
      "materialImage": "/uploads/materials/1234567890.jpg",
      "remarks": "비고",
      "isActive": true,
      "createdAt": "2026-02-20T08:00:00.000Z"
    }
  ],
  "total": 100,
  "page": 1,
  "limit": 10
}
```

---

### 3.2 안전재고 미달 자재 목록
```
GET /materials/low-stock
```
🔒 **인증 필요**

**Response (200):**
```json
[
  {
    "id": 1,
    "materialCode": "MAT001",
    "materialName": "IP 카메라",
    "currentStock": 3,
    "safetyStockLevel": 10,
    "reorderPoint": 5,
    "shortage": 7
  }
]
```

---

### 3.3 자재 상세 조회
```
GET /materials/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "materialCode": "MAT001",
  "materialName": "IP 카메라",
  "categoryId": 1,
  "category": {
    "id": 1,
    "categoryName": "보안장비"
  },
  "manufacturer": "Hikvision",
  "modelNumber": "DS-2CD2143G0-I",
  "specifications": {
    "resolution": "4MP",
    "lens": "2.8mm"
  },
  "unit": "EA",
  "unitPrice": 150000,
  "safetyStockLevel": 10,
  "reorderPoint": 5,
  "requiresSerial": false,
  "isBomProduct": false,
  "materialImage": "/uploads/materials/1234567890.jpg",
  "remarks": "비고",
  "isActive": true,
  "createdAt": "2026-02-20T08:00:00.000Z",
  "updatedAt": "2026-02-20T08:00:00.000Z"
}
```

**Error (404):** 자재를 찾을 수 없음

---

### 3.4 자재 생성
```
POST /materials
```
🔒 **인증 필요**  
📎 **multipart/form-data**

**Request Body (Form Data):**
- `materialCode`: string (필수, 자재 코드)
- `materialName`: string (필수, 자재명)
- `categoryId`: number (필수, 카테고리 ID)
- `manufacturer`: string (선택, 제조사)
- `modelNumber`: string (선택, 모델번호)
- `specifications`: string (선택, JSON 문자열)
- `unit`: string (선택, 단위, 기본값: "EA")
- `unitPrice`: number (선택, 단가)
- `safetyStockLevel`: number (선택, 안전재고)
- `reorderPoint`: number (선택, 재주문점)
- `requiresSerial`: boolean (선택, 시리얼 관리 여부)
- `isBomProduct`: boolean (선택, BOM 제품 여부)
- `remarks`: string (선택, 비고)
- `image`: file (선택, JPEG/PNG/GIF, 최대 5MB)

**Response (201):**
```json
{
  "id": 1,
  "materialCode": "MAT001",
  "materialName": "IP 카메라",
  "categoryId": 1,
  "materialImage": "/uploads/materials/1234567890.jpg",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

**Error (400):** 중복된 자재 코드 또는 잘못된 요청

---

### 3.5 자재 수정
```
PATCH /materials/:id
```
🔒 **인증 필요**  
📎 **multipart/form-data**

**Request Body (Form Data):** 3.4와 동일 (모든 필드 선택)

**Response (200):**
```json
{
  "id": 1,
  "materialCode": "MAT001",
  "materialName": "IP 카메라 (수정)",
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

**Error (404):** 자재를 찾을 수 없음

---

### 3.6 자재 삭제
```
DELETE /materials/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "message": "삭제 성공"
}
```

**Error (404):** 자재를 찾을 수 없음

---

## 4. 자재 카테고리 (Material Categories)

### 4.1 카테고리 목록 조회
```
GET /material-categories
```
🔒 **인증 필요**

**Response (200):**
```json
[
  {
    "id": 1,
    "categoryCode": "CAT001",
    "categoryName": "보안장비",
    "parentId": null,
    "description": "CCTV, 출입통제 등",
    "isActive": true,
    "children": [
      {
        "id": 2,
        "categoryCode": "CAT001-01",
        "categoryName": "CCTV",
        "parentId": 1,
        "isActive": true
      }
    ]
  }
]
```

---

### 4.2 카테고리 상세 조회
```
GET /material-categories/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "categoryCode": "CAT001",
  "categoryName": "보안장비",
  "parentId": null,
  "description": "CCTV, 출입통제 등",
  "isActive": true,
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

**Error (404):** 카테고리를 찾을 수 없음

---

### 4.3 카테고리 생성
```
POST /material-categories
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "categoryCode": "CAT001",
  "categoryName": "보안장비",
  "parentId": null,
  "description": "CCTV, 출입통제 등"
}
```

**Response (201):**
```json
{
  "id": 1,
  "categoryCode": "CAT001",
  "categoryName": "보안장비",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

---

### 4.4 카테고리 수정
```
PATCH /material-categories/:id
```
🔒 **인증 필요**

**Request Body:** 4.3과 동일 (모든 필드 선택)

**Response (200):**
```json
{
  "id": 1,
  "categoryName": "보안장비 (수정)",
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

**Error (404):** 카테고리를 찾을 수 없음

---

### 4.5 카테고리 삭제
```
DELETE /material-categories/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "message": "삭제 성공"
}
```

**Error (404):** 카테고리를 찾을 수 없음

---

## 5. BOM 관리 (BOM Masters)

### 5.1 BOM 목록 조회
```
GET /bom-masters
```
🔒 **인증 필요**

**Response (200):**
```json
[
  {
    "id": 1,
    "bomCode": "BOM001",
    "bomName": "IP 카메라 세트",
    "productMaterialId": 1,
    "productMaterial": {
      "id": 1,
      "materialCode": "MAT001",
      "materialName": "IP 카메라 세트"
    },
    "version": "1.0",
    "isActive": true,
    "createdAt": "2026-02-20T08:00:00.000Z"
  }
]
```

---

### 5.2 BOM 상세 조회 (구성 자재 포함)
```
GET /bom-masters/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "bomCode": "BOM001",
  "bomName": "IP 카메라 세트",
  "productMaterialId": 1,
  "version": "1.0",
  "isActive": true,
  "items": [
    {
      "id": 1,
      "bomMasterId": 1,
      "materialId": 2,
      "material": {
        "id": 2,
        "materialCode": "MAT002",
        "materialName": "카메라 본체"
      },
      "quantity": 1,
      "unit": "EA",
      "remarks": "메인 구성품"
    },
    {
      "id": 2,
      "bomMasterId": 1,
      "materialId": 3,
      "material": {
        "id": 3,
        "materialCode": "MAT003",
        "materialName": "전원 어댑터"
      },
      "quantity": 1,
      "unit": "EA"
    }
  ],
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

**Error (404):** BOM을 찾을 수 없음

---

### 5.3 BOM 생성
```
POST /bom-masters
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "bomCode": "BOM001",
  "bomName": "IP 카메라 세트",
  "productMaterialId": 1,
  "version": "1.0",
  "description": "표준 IP 카메라 세트"
}
```

**Response (201):**
```json
{
  "id": 1,
  "bomCode": "BOM001",
  "bomName": "IP 카메라 세트",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

---

### 5.4 BOM 수정
```
PATCH /bom-masters/:id
```
🔒 **인증 필요**

**Request Body:** 5.3과 동일 (모든 필드 선택)

**Response (200):**
```json
{
  "id": 1,
  "bomName": "IP 카메라 세트 (수정)",
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

**Error (404):** BOM을 찾을 수 없음

---

### 5.5 BOM 삭제
```
DELETE /bom-masters/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "message": "삭제 성공"
}
```

**Error (404):** BOM을 찾을 수 없음

---

### 5.6 BOM 구성 자재 추가
```
POST /bom-masters/:id/items
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "materialId": 2,
  "quantity": 1,
  "unit": "EA",
  "remarks": "메인 구성품"
}
```

**Response (201):**
```json
{
  "id": 1,
  "bomMasterId": 1,
  "materialId": 2,
  "quantity": 1,
  "unit": "EA",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

---

### 5.7 BOM 전개 (소요량 계산)
```
POST /bom-masters/:id/explode?quantity=10
```
🔒 **인증 필요**

**Query Parameters:**
- `quantity`: number (필수, 생산 수량)

**Response (200):**
```json
{
  "bomId": 1,
  "bomName": "IP 카메라 세트",
  "productionQuantity": 10,
  "requiredMaterials": [
    {
      "materialId": 2,
      "materialCode": "MAT002",
      "materialName": "카메라 본체",
      "requiredQuantity": 10,
      "unit": "EA",
      "currentStock": 15,
      "shortage": 0
    },
    {
      "materialId": 3,
      "materialCode": "MAT003",
      "materialName": "전원 어댑터",
      "requiredQuantity": 10,
      "unit": "EA",
      "currentStock": 5,
      "shortage": 5
    }
  ]
}
```

---

### 5.8 BOM 구성 자재 수정
```
PATCH /bom-items/:id
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "quantity": 2,
  "remarks": "수정된 비고"
}
```

**Response (200):**
```json
{
  "id": 1,
  "quantity": 2,
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

**Error (404):** BOM 항목을 찾을 수 없음

---

### 5.9 BOM 구성 자재 삭제
```
DELETE /bom-items/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "message": "삭제 성공"
}
```

**Error (404):** BOM 항목을 찾을 수 없음

---
## 6. 현장 관리 (Sites)

### 6.1 현장 목록 조회
```
GET /sites
```
🔒 **인증 필요**

**Response (200):**
```json
[
  {
    "id": 1,
    "siteCode": "SITE001",
    "siteName": "강남 오피스텔 신축",
    "location": "서울시 강남구",
    "startDate": "2026-01-01",
    "endDate": "2026-12-31",
    "status": "IN_PROGRESS",
    "managerName": "김현장",
    "managerPhone": "010-1234-5678",
    "createdAt": "2026-02-20T08:00:00.000Z"
  }
]
```

---

### 6.2 현장 상세 조회
```
GET /sites/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "siteCode": "SITE001",
  "siteName": "강남 오피스텔 신축",
  "location": "서울시 강남구 테헤란로 123",
  "startDate": "2026-01-01",
  "endDate": "2026-12-31",
  "status": "IN_PROGRESS",
  "managerName": "김현장",
  "managerPhone": "010-1234-5678",
  "managerEmail": "manager@example.com",
  "description": "30층 규모 오피스텔",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

**Error (404):** 현장을 찾을 수 없음

---

### 6.3 현장 상태 변경
```
PATCH /sites/:id/status
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "status": "COMPLETED"
}
```

**상태 값:**
- `PLANNING`: 계획
- `IN_PROGRESS`: 진행중
- `COMPLETED`: 완료
- `SUSPENDED`: 중단

**Response (200):**
```json
{
  "id": 1,
  "status": "COMPLETED",
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

---

### 6.4 현장 견적 목록 조회
```
GET /sites/:id/quotations
```
🔒 **인증 필요**

**Response (200):**
```json
[
  {
    "id": 1,
    "quotationNumber": "Q-2026-001",
    "siteId": 1,
    "totalAmount": 50000000,
    "status": "APPROVED",
    "createdAt": "2026-02-20T08:00:00.000Z"
  }
]
```

---

### 6.5 현장 자재 소요/납품 현황
```
GET /sites/:id/materials-status
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "siteId": 1,
  "siteName": "강남 오피스텔 신축",
  "materials": [
    {
      "materialId": 1,
      "materialCode": "MAT001",
      "materialName": "IP 카메라",
      "requiredQuantity": 100,
      "deliveredQuantity": 80,
      "remainingQuantity": 20,
      "deliveryRate": 80
    }
  ]
}
```

---

## 7. 창고 관리 (Warehouses)

### 7.1 창고 목록 조회
```
GET /warehouses
```
🔒 **인증 필요**

**Response (200):**
```json
[
  {
    "id": 1,
    "warehouseCode": "WH001",
    "warehouseName": "본사 창고",
    "location": "서울시 강남구",
    "managerName": "이창고",
    "managerPhone": "010-2345-6789",
    "isActive": true,
    "createdAt": "2026-02-20T08:00:00.000Z"
  }
]
```

---

### 7.2 창고 상세 조회
```
GET /warehouses/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "warehouseCode": "WH001",
  "warehouseName": "본사 창고",
  "location": "서울시 강남구 테헤란로 456",
  "managerName": "이창고",
  "managerPhone": "010-2345-6789",
  "managerEmail": "warehouse@example.com",
  "capacity": 1000,
  "isActive": true,
  "description": "메인 보관 창고",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

**Error (404):** 창고를 찾을 수 없음

---

### 7.3 창고 생성
```
POST /warehouses
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "warehouseCode": "WH001",
  "warehouseName": "본사 창고",
  "location": "서울시 강남구 테헤란로 456",
  "managerName": "이창고",
  "managerPhone": "010-2345-6789",
  "managerEmail": "warehouse@example.com",
  "capacity": 1000,
  "description": "메인 보관 창고"
}
```

**Response (201):**
```json
{
  "id": 1,
  "warehouseCode": "WH001",
  "warehouseName": "본사 창고",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

---

### 7.4 창고 수정
```
PATCH /warehouses/:id
```
🔒 **인증 필요**

**Request Body:** 7.3과 동일 (모든 필드 선택)

**Response (200):**
```json
{
  "id": 1,
  "warehouseName": "본사 창고 (수정)",
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

**Error (404):** 창고를 찾을 수 없음

---

### 7.5 창고 삭제
```
DELETE /warehouses/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "message": "삭제 성공"
}
```

**Error (404):** 창고를 찾을 수 없음

---

### 7.6 창고 로케이션 목록 조회
```
GET /warehouses/:id/locations
```
🔒 **인증 필요**

**Response (200):**
```json
[
  {
    "id": 1,
    "warehouseId": 1,
    "locationCode": "A-01-01",
    "locationName": "A동 1층 1번",
    "zone": "A",
    "aisle": "01",
    "rack": "01",
    "level": "01",
    "isActive": true,
    "createdAt": "2026-02-20T08:00:00.000Z"
  }
]
```

---

### 7.7 창고 로케이션 추가
```
POST /warehouses/:id/locations
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "locationCode": "A-01-01",
  "locationName": "A동 1층 1번",
  "zone": "A",
  "aisle": "01",
  "rack": "01",
  "level": "01"
}
```

**Response (201):**
```json
{
  "id": 1,
  "warehouseId": 1,
  "locationCode": "A-01-01",
  "locationName": "A동 1층 1번",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

---

### 7.8 창고 로케이션 수정
```
PATCH /warehouses/locations/:id
```
🔒 **인증 필요**

**Request Body:** 7.7과 동일 (모든 필드 선택)

**Response (200):**
```json
{
  "id": 1,
  "locationName": "A동 1층 1번 (수정)",
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

**Error (404):** 로케이션을 찾을 수 없음

---

### 7.9 창고 로케이션 삭제
```
DELETE /warehouses/locations/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "message": "삭제 성공"
}
```

**Error (404):** 로케이션을 찾을 수 없음

---

## 8. 공급사 관리 (Suppliers)

### 8.1 공급사 목록 조회
```
GET /suppliers
```
🔒 **인증 필요**

**Response (200):**
```json
[
  {
    "id": 1,
    "supplierCode": "SUP001",
    "supplierName": "한국전자",
    "businessNumber": "123-45-67890",
    "representative": "박대표",
    "contactPerson": "김담당",
    "phone": "02-1234-5678",
    "email": "contact@supplier.com",
    "address": "서울시 강남구",
    "isActive": true,
    "createdAt": "2026-02-20T08:00:00.000Z"
  }
]
```

---

### 8.2 공급사 상세 조회
```
GET /suppliers/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "supplierCode": "SUP001",
  "supplierName": "한국전자",
  "businessNumber": "123-45-67890",
  "representative": "박대표",
  "contactPerson": "김담당",
  "phone": "02-1234-5678",
  "email": "contact@supplier.com",
  "address": "서울시 강남구 테헤란로 789",
  "paymentTerms": "월말 결제",
  "deliveryTerms": "주문 후 3일",
  "remarks": "주요 공급사",
  "isActive": true,
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

**Error (404):** 공급사를 찾을 수 없음

---

### 8.3 공급사 생성
```
POST /suppliers
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "supplierCode": "SUP001",
  "supplierName": "한국전자",
  "businessNumber": "123-45-67890",
  "representative": "박대표",
  "contactPerson": "김담당",
  "phone": "02-1234-5678",
  "email": "contact@supplier.com",
  "address": "서울시 강남구 테헤란로 789",
  "paymentTerms": "월말 결제",
  "deliveryTerms": "주문 후 3일",
  "remarks": "주요 공급사"
}
```

**Response (201):**
```json
{
  "id": 1,
  "supplierCode": "SUP001",
  "supplierName": "한국전자",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

---

### 8.4 공급사 수정
```
PATCH /suppliers/:id
```
🔒 **인증 필요**

**Request Body:** 8.3과 동일 (모든 필드 선택)

**Response (200):**
```json
{
  "id": 1,
  "supplierName": "한국전자 (수정)",
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

**Error (404):** 공급사를 찾을 수 없음

---

### 8.5 공급사 삭제
```
DELETE /suppliers/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "message": "삭제 성공"
}
```

**Error (404):** 공급사를 찾을 수 없음

---

### 8.6 공급사 발주 목록 조회
```
GET /suppliers/:id/purchase-orders
```
🔒 **인증 필요**

**Response (200):**
```json
[
  {
    "id": 1,
    "poNumber": "PO-2026-001",
    "supplierId": 1,
    "totalAmount": 10000000,
    "status": "APPROVED",
    "orderDate": "2026-02-20",
    "createdAt": "2026-02-20T08:00:00.000Z"
  }
]
```

---

## 9. 재고 관리 (Stocks)

### 9.1 재고 목록 조회
```
GET /stocks
```
🔒 **인증 필요**

**Query Parameters:**
- `page`: number (선택, 기본값: 1)
- `limit`: number (선택, 기본값: 10)
- `materialId`: number (선택, 자재 필터)
- `locationId`: number (선택, 로케이션 필터)
- `warehouseId`: number (선택, 창고 필터)

**Response (200):**
```json
{
  "data": [
    {
      "id": 1,
      "materialId": 1,
      "material": {
        "id": 1,
        "materialCode": "MAT001",
        "materialName": "IP 카메라"
      },
      "locationId": 1,
      "location": {
        "id": 1,
        "locationCode": "A-01-01",
        "locationName": "A동 1층 1번",
        "warehouseId": 1,
        "warehouse": {
          "id": 1,
          "warehouseName": "본사 창고"
        }
      },
      "quantity": 50,
      "lotNumber": "LOT-2026-001",
      "expiryDate": "2027-12-31",
      "createdAt": "2026-02-20T08:00:00.000Z"
    }
  ],
  "total": 100,
  "page": 1,
  "limit": 10
}
```

---

### 9.2 재고 요약 (자재별 총 재고)
```
GET /stocks/summary
```
🔒 **인증 필요**

**Response (200):**
```json
[
  {
    "materialId": 1,
    "materialCode": "MAT001",
    "materialName": "IP 카메라",
    "totalQuantity": 150,
    "unit": "EA",
    "safetyStockLevel": 10,
    "reorderPoint": 5,
    "status": "SUFFICIENT"
  }
]
```

**상태 값:**
- `SUFFICIENT`: 충분
- `LOW`: 재주문점 미달
- `CRITICAL`: 안전재고 미달

---

### 9.3 유효기한 임박 재고
```
GET /stocks/expiring?days=30
```
🔒 **인증 필요**

**Query Parameters:**
- `days`: number (선택, 기본값: 30, 임박 일수)

**Response (200):**
```json
[
  {
    "id": 1,
    "materialId": 1,
    "materialCode": "MAT001",
    "materialName": "IP 카메라",
    "locationCode": "A-01-01",
    "quantity": 10,
    "lotNumber": "LOT-2026-001",
    "expiryDate": "2026-03-15",
    "daysUntilExpiry": 23
  }
]
```

---

### 9.4 재고 상세 조회
```
GET /stocks/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "materialId": 1,
  "material": {
    "id": 1,
    "materialCode": "MAT001",
    "materialName": "IP 카메라",
    "unit": "EA"
  },
  "locationId": 1,
  "location": {
    "id": 1,
    "locationCode": "A-01-01",
    "locationName": "A동 1층 1번",
    "warehouseId": 1,
    "warehouse": {
      "id": 1,
      "warehouseName": "본사 창고"
    }
  },
  "quantity": 50,
  "lotNumber": "LOT-2026-001",
  "expiryDate": "2027-12-31",
  "createdAt": "2026-02-20T08:00:00.000Z",
  "updatedAt": "2026-02-20T08:00:00.000Z"
}
```

**Error (404):** 재고를 찾을 수 없음

---
## 10. 재고 이동 (Stock Movements)

### 10.1 재고 이동 목록 조회
```
GET /stock-movements
```
🔒 **인증 필요**

**Query Parameters:**
- `movementType`: string (선택, 이동 유형 필터)
- `status`: string (선택, 상태 필터)

**이동 유형:**
- `PURCHASE_IN`: 구매 입고
- `SITE_OUT`: 현장 출고
- `RETURN_IN`: 현장 반품 입고
- `RETURN_OUT`: 공급사 반품 출고
- `TRANSFER`: 창고 간 이동

**상태:**
- `PENDING`: 대기
- `APPROVED`: 승인
- `CANCELLED`: 취소

**Response (200):**
```json
[
  {
    "id": 1,
    "movementNumber": "MV-2026-001",
    "movementType": "PURCHASE_IN",
    "materialId": 1,
    "material": {
      "id": 1,
      "materialCode": "MAT001",
      "materialName": "IP 카메라"
    },
    "fromLocationId": null,
    "toLocationId": 1,
    "toLocation": {
      "id": 1,
      "locationCode": "A-01-01",
      "warehouseName": "본사 창고"
    },
    "quantity": 50,
    "status": "APPROVED",
    "movementDate": "2026-02-20",
    "createdAt": "2026-02-20T08:00:00.000Z"
  }
]
```

---

### 10.2 재고 이동 상세 조회
```
GET /stock-movements/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "movementNumber": "MV-2026-001",
  "movementType": "PURCHASE_IN",
  "materialId": 1,
  "material": {
    "id": 1,
    "materialCode": "MAT001",
    "materialName": "IP 카메라"
  },
  "fromLocationId": null,
  "toLocationId": 1,
  "toLocation": {
    "id": 1,
    "locationCode": "A-01-01",
    "locationName": "A동 1층 1번",
    "warehouseId": 1,
    "warehouse": {
      "id": 1,
      "warehouseName": "본사 창고"
    }
  },
  "quantity": 50,
  "lotNumber": "LOT-2026-001",
  "expiryDate": "2027-12-31",
  "status": "APPROVED",
  "movementDate": "2026-02-20",
  "remarks": "정상 입고",
  "createdBy": 1,
  "approvedBy": 2,
  "approvedAt": "2026-02-20T09:00:00.000Z",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

**Error (404):** 재고 이동을 찾을 수 없음

---

### 10.3 구매 입고
```
POST /stock-movements/purchase-in
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "materialId": 1,
  "toLocationId": 1,
  "quantity": 50,
  "lotNumber": "LOT-2026-001",
  "expiryDate": "2027-12-31",
  "movementDate": "2026-02-20",
  "remarks": "정상 입고"
}
```

**Response (201):**
```json
{
  "id": 1,
  "movementNumber": "MV-2026-001",
  "movementType": "PURCHASE_IN",
  "status": "PENDING",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

---

### 10.4 현장 출고
```
POST /stock-movements/site-out
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "materialId": 1,
  "fromLocationId": 1,
  "quantity": 10,
  "siteId": 1,
  "movementDate": "2026-02-20",
  "remarks": "현장 납품"
}
```

**Response (201):**
```json
{
  "id": 2,
  "movementNumber": "MV-2026-002",
  "movementType": "SITE_OUT",
  "status": "PENDING",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

---

### 10.5 현장 반품 입고
```
POST /stock-movements/return-in
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "materialId": 1,
  "toLocationId": 1,
  "quantity": 2,
  "siteId": 1,
  "movementDate": "2026-02-20",
  "remarks": "현장 반품"
}
```

**Response (201):**
```json
{
  "id": 3,
  "movementNumber": "MV-2026-003",
  "movementType": "RETURN_IN",
  "status": "PENDING",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

---

### 10.6 공급사 반품 출고
```
POST /stock-movements/return-out
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "materialId": 1,
  "fromLocationId": 1,
  "quantity": 5,
  "supplierId": 1,
  "movementDate": "2026-02-20",
  "remarks": "불량품 반품"
}
```

**Response (201):**
```json
{
  "id": 4,
  "movementNumber": "MV-2026-004",
  "movementType": "RETURN_OUT",
  "status": "PENDING",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

---

### 10.7 창고 간 이동
```
POST /stock-movements/transfer
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "materialId": 1,
  "fromLocationId": 1,
  "toLocationId": 2,
  "quantity": 20,
  "movementDate": "2026-02-20",
  "remarks": "창고 이동"
}
```

**Response (201):**
```json
{
  "id": 5,
  "movementNumber": "MV-2026-005",
  "movementType": "TRANSFER",
  "status": "PENDING",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

---

### 10.8 재고 이동 승인
```
PATCH /stock-movements/:id/approve
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "status": "APPROVED",
  "approvedBy": 2,
  "approvedAt": "2026-02-20T09:00:00.000Z"
}
```

**Error (400):** 승인 불가 (이미 승인됨 또는 취소됨)

---

### 10.9 재고 이동 취소
```
PATCH /stock-movements/:id/cancel
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "status": "CANCELLED",
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

**Error (400):** 취소 불가 (이미 승인됨)

---

## 11. 재고 조정 (Stock Adjustments)

### 11.1 재고 조정 목록 조회
```
GET /stock-adjustments
```
🔒 **인증 필요**

**Query Parameters:**
- `approved`: boolean (선택, 승인 여부 필터)

**Response (200):**
```json
[
  {
    "id": 1,
    "adjustmentNumber": "ADJ-2026-001",
    "materialId": 1,
    "material": {
      "id": 1,
      "materialCode": "MAT001",
      "materialName": "IP 카메라"
    },
    "locationId": 1,
    "location": {
      "id": 1,
      "locationCode": "A-01-01",
      "warehouseName": "본사 창고"
    },
    "beforeQuantity": 50,
    "afterQuantity": 48,
    "adjustmentQuantity": -2,
    "reason": "재고 실사",
    "isApproved": true,
    "adjustmentDate": "2026-02-20",
    "createdAt": "2026-02-20T08:00:00.000Z"
  }
]
```

---

### 11.2 재고 조정 상세 조회
```
GET /stock-adjustments/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "adjustmentNumber": "ADJ-2026-001",
  "materialId": 1,
  "material": {
    "id": 1,
    "materialCode": "MAT001",
    "materialName": "IP 카메라"
  },
  "locationId": 1,
  "location": {
    "id": 1,
    "locationCode": "A-01-01",
    "locationName": "A동 1층 1번",
    "warehouseId": 1,
    "warehouse": {
      "id": 1,
      "warehouseName": "본사 창고"
    }
  },
  "beforeQuantity": 50,
  "afterQuantity": 48,
  "adjustmentQuantity": -2,
  "reason": "재고 실사",
  "remarks": "파손 2개 발견",
  "isApproved": true,
  "adjustmentDate": "2026-02-20",
  "createdBy": 1,
  "approvedBy": 2,
  "approvedAt": "2026-02-20T09:00:00.000Z",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

**Error (404):** 재고 조정을 찾을 수 없음

---

### 11.3 재고 조정 생성
```
POST /stock-adjustments
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "materialId": 1,
  "locationId": 1,
  "afterQuantity": 48,
  "reason": "재고 실사",
  "remarks": "파손 2개 발견",
  "adjustmentDate": "2026-02-20"
}
```

**Response (201):**
```json
{
  "id": 1,
  "adjustmentNumber": "ADJ-2026-001",
  "beforeQuantity": 50,
  "afterQuantity": 48,
  "adjustmentQuantity": -2,
  "isApproved": false,
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

---

### 11.4 재고 조정 수정
```
PATCH /stock-adjustments/:id
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "afterQuantity": 47,
  "remarks": "파손 3개 발견 (수정)"
}
```

**Response (200):**
```json
{
  "id": 1,
  "afterQuantity": 47,
  "adjustmentQuantity": -3,
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

**Error (400):** 수정 불가 (이미 승인됨)

---

### 11.5 재고 조정 삭제
```
DELETE /stock-adjustments/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "message": "삭제 성공"
}
```

**Error (400):** 삭제 불가 (이미 승인됨)

---

### 11.6 재고 조정 승인
```
PATCH /stock-adjustments/:id/approve
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "isApproved": true,
  "approvedBy": 2,
  "approvedAt": "2026-02-20T09:00:00.000Z"
}
```

**Error (400):** 승인 불가 (이미 승인됨)

---

## 12. 견적 관리 (Quotations)

### 12.1 견적 목록 조회
```
GET /quotations
```
🔒 **인증 필요**

**Query Parameters:**
- `status`: string (선택, 상태 필터)

**상태:**
- `DRAFT`: 작성중
- `SUBMITTED`: 제출
- `APPROVED`: 승인
- `REJECTED`: 거절
- `EXPIRED`: 만료

**Response (200):**
```json
[
  {
    "id": 1,
    "quotationNumber": "Q-2026-001",
    "siteId": 1,
    "site": {
      "id": 1,
      "siteCode": "SITE001",
      "siteName": "강남 오피스텔 신축"
    },
    "totalAmount": 50000000,
    "status": "APPROVED",
    "validUntil": "2026-03-31",
    "isSigned": true,
    "signedAt": "2026-02-20T10:00:00.000Z",
    "createdAt": "2026-02-20T08:00:00.000Z"
  }
]
```

---

### 12.2 견적 상세 조회
```
GET /quotations/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "quotationNumber": "Q-2026-001",
  "siteId": 1,
  "site": {
    "id": 1,
    "siteCode": "SITE001",
    "siteName": "강남 오피스텔 신축"
  },
  "totalAmount": 50000000,
  "status": "APPROVED",
  "validUntil": "2026-03-31",
  "remarks": "표준 견적",
  "isSigned": true,
  "signedBy": 1,
  "signedAt": "2026-02-20T10:00:00.000Z",
  "createdBy": 1,
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

**Error (404):** 견적을 찾을 수 없음

---

### 12.3 견적 생성
```
POST /quotations
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "siteId": 1,
  "validUntil": "2026-03-31",
  "remarks": "표준 견적"
}
```

**Response (201):**
```json
{
  "id": 1,
  "quotationNumber": "Q-2026-001",
  "status": "DRAFT",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

---

### 12.4 견적 수정
```
PATCH /quotations/:id
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "validUntil": "2026-04-30",
  "remarks": "표준 견적 (수정)"
}
```

**Response (200):**
```json
{
  "id": 1,
  "validUntil": "2026-04-30",
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

**Error (400):** 수정 불가 (승인됨 또는 서명됨)

---

### 12.5 견적 삭제
```
DELETE /quotations/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "message": "삭제 성공"
}
```

**Error (400):** 삭제 불가 (승인됨 또는 서명됨)

---

### 12.6 견적 서명
```
PATCH /quotations/:id/sign
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "isSigned": true,
  "signedBy": 1,
  "signedAt": "2026-02-20T10:00:00.000Z"
}
```

**Error (400):** 서명 불가 (이미 서명됨)

---

### 12.7 견적 상태 변경
```
PATCH /quotations/:id/status
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "status": "APPROVED"
}
```

**Response (200):**
```json
{
  "id": 1,
  "status": "APPROVED",
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

---

### 12.8 견적 자재 목록 조회
```
GET /quotations/:id/materials
```
🔒 **인증 필요**

**Response (200):**
```json
[
  {
    "id": 1,
    "quotationId": 1,
    "materialId": 1,
    "material": {
      "id": 1,
      "materialCode": "MAT001",
      "materialName": "IP 카메라"
    },
    "quantity": 100,
    "unitPrice": 150000,
    "totalPrice": 15000000,
    "status": "PENDING",
    "deliveredQuantity": 0,
    "remarks": "표준 모델"
  }
]
```

---

### 12.9 견적 자재 추가
```
POST /quotations/:id/materials
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "materialId": 1,
  "quantity": 100,
  "unitPrice": 150000,
  "remarks": "표준 모델"
}
```

**Response (201):**
```json
{
  "id": 1,
  "quotationId": 1,
  "materialId": 1,
  "quantity": 100,
  "unitPrice": 150000,
  "totalPrice": 15000000,
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

**Error (400):** 추가 불가 (견적이 승인됨)

---

### 12.10 견적 자재 수정
```
PATCH /quotations/materials/:id
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "quantity": 120,
  "unitPrice": 145000
}
```

**Response (200):**
```json
{
  "id": 1,
  "quantity": 120,
  "unitPrice": 145000,
  "totalPrice": 17400000,
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

**Error (400):** 수정 불가 (견적이 승인됨)

---

### 12.11 견적 자재 삭제
```
DELETE /quotations/materials/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "message": "삭제 성공"
}
```

**Error (400):** 삭제 불가 (견적이 승인됨)

---

### 12.12 견적 자재 상태 변경
```
PATCH /quotations/materials/:id/status
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "status": "DELIVERED"
}
```

**상태 값:**
- `PENDING`: 대기
- `ORDERED`: 발주됨
- `DELIVERED`: 납품완료

**Response (200):**
```json
{
  "id": 1,
  "status": "DELIVERED",
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

---
## 13. 발주 관리 (Purchase Orders)

### 13.1 발주 목록 조회
```
GET /purchase-orders
```
🔒 **인증 필요**

**Query Parameters:**
- `status`: string (선택, 상태 필터)

**상태:**
- `DRAFT`: 작성중
- `SUBMITTED`: 제출
- `APPROVED`: 승인
- `ORDERED`: 발주완료
- `RECEIVED`: 입고완료
- `CANCELLED`: 취소

**Response (200):**
```json
[
  {
    "id": 1,
    "poNumber": "PO-2026-001",
    "supplierId": 1,
    "supplier": {
      "id": 1,
      "supplierCode": "SUP001",
      "supplierName": "한국전자"
    },
    "totalAmount": 10000000,
    "status": "APPROVED",
    "orderDate": "2026-02-20",
    "expectedDeliveryDate": "2026-02-27",
    "createdAt": "2026-02-20T08:00:00.000Z"
  }
]
```

---

### 13.2 발주 상세 조회
```
GET /purchase-orders/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "poNumber": "PO-2026-001",
  "supplierId": 1,
  "supplier": {
    "id": 1,
    "supplierCode": "SUP001",
    "supplierName": "한국전자",
    "contactPerson": "김담당",
    "phone": "02-1234-5678"
  },
  "totalAmount": 10000000,
  "status": "APPROVED",
  "orderDate": "2026-02-20",
  "expectedDeliveryDate": "2026-02-27",
  "remarks": "긴급 발주",
  "createdBy": 1,
  "approvedBy": 2,
  "approvedAt": "2026-02-20T09:00:00.000Z",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

**Error (404):** 발주를 찾을 수 없음

---

### 13.3 발주 생성
```
POST /purchase-orders
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "supplierId": 1,
  "orderDate": "2026-02-20",
  "expectedDeliveryDate": "2026-02-27",
  "remarks": "긴급 발주"
}
```

**Response (201):**
```json
{
  "id": 1,
  "poNumber": "PO-2026-001",
  "status": "DRAFT",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

---

### 13.4 발주 수정
```
PATCH /purchase-orders/:id
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "expectedDeliveryDate": "2026-02-28",
  "remarks": "긴급 발주 (수정)"
}
```

**Response (200):**
```json
{
  "id": 1,
  "expectedDeliveryDate": "2026-02-28",
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

**Error (400):** 수정 불가 (승인됨 또는 발주완료)

---

### 13.5 발주 삭제
```
DELETE /purchase-orders/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "message": "삭제 성공"
}
```

**Error (400):** 삭제 불가 (승인됨 또는 발주완료)

---

### 13.6 발주 제출
```
PATCH /purchase-orders/:id/submit
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "status": "SUBMITTED",
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

**Error (400):** 제출 불가 (품목이 없음 또는 이미 제출됨)

---

### 13.7 발주 승인
```
PATCH /purchase-orders/:id/approve
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "status": "APPROVED",
  "approvedBy": 2,
  "approvedAt": "2026-02-20T09:00:00.000Z"
}
```

**Error (400):** 승인 불가 (제출되지 않음 또는 이미 승인됨)

---

### 13.8 발주 취소
```
PATCH /purchase-orders/:id/cancel
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "status": "CANCELLED",
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

**Error (400):** 취소 불가 (이미 입고완료)

---

### 13.9 발주 품목 목록 조회
```
GET /purchase-orders/:id/items
```
🔒 **인증 필요**

**Response (200):**
```json
[
  {
    "id": 1,
    "purchaseOrderId": 1,
    "materialId": 1,
    "material": {
      "id": 1,
      "materialCode": "MAT001",
      "materialName": "IP 카메라"
    },
    "quantity": 50,
    "unitPrice": 150000,
    "totalPrice": 7500000,
    "receivedQuantity": 0,
    "remarks": "표준 모델"
  }
]
```

---

### 13.10 발주 품목 추가
```
POST /purchase-orders/:id/items
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "materialId": 1,
  "quantity": 50,
  "unitPrice": 150000,
  "remarks": "표준 모델"
}
```

**Response (201):**
```json
{
  "id": 1,
  "purchaseOrderId": 1,
  "materialId": 1,
  "quantity": 50,
  "unitPrice": 150000,
  "totalPrice": 7500000,
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

**Error (400):** 추가 불가 (발주가 승인됨)

---

### 13.11 발주 품목 수정
```
PATCH /purchase-orders/items/:id
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "quantity": 60,
  "unitPrice": 145000
}
```

**Response (200):**
```json
{
  "id": 1,
  "quantity": 60,
  "unitPrice": 145000,
  "totalPrice": 8700000,
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

**Error (400):** 수정 불가 (발주가 승인됨)

---

### 13.12 발주 품목 삭제
```
DELETE /purchase-orders/items/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "message": "삭제 성공"
}
```

**Error (400):** 삭제 불가 (발주가 승인됨)

---

### 13.13 발주 품목 입고
```
PATCH /purchase-orders/items/:id/receive
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "receivedQuantity": 50
}
```

**Response (200):**
```json
{
  "id": 1,
  "receivedQuantity": 50,
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

---

## 14. 시리얼 번호 관리 (Material Serials)

### 14.1 시리얼 번호 목록 조회
```
GET /material-serials
```
🔒 **인증 필요**

**Query Parameters:**
- `status`: string (선택, 상태 필터)
- `materialId`: number (선택, 자재 필터)

**상태:**
- `IN_STOCK`: 재고
- `IN_USE`: 사용중
- `DEFECTIVE`: 불량
- `RETURNED`: 반품

**Response (200):**
```json
[
  {
    "id": 1,
    "serialNumber": "SN-2026-001",
    "materialId": 1,
    "material": {
      "id": 1,
      "materialCode": "MAT001",
      "materialName": "IP 카메라"
    },
    "locationId": 1,
    "location": {
      "id": 1,
      "locationCode": "A-01-01",
      "warehouseName": "본사 창고"
    },
    "status": "IN_STOCK",
    "manufactureDate": "2026-01-15",
    "warrantyPeriod": 24,
    "createdAt": "2026-02-20T08:00:00.000Z"
  }
]
```

---

### 14.2 시리얼 번호 상세 조회
```
GET /material-serials/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "id": 1,
  "serialNumber": "SN-2026-001",
  "materialId": 1,
  "material": {
    "id": 1,
    "materialCode": "MAT001",
    "materialName": "IP 카메라"
  },
  "locationId": 1,
  "location": {
    "id": 1,
    "locationCode": "A-01-01",
    "locationName": "A동 1층 1번",
    "warehouseId": 1,
    "warehouse": {
      "id": 1,
      "warehouseName": "본사 창고"
    }
  },
  "status": "IN_STOCK",
  "manufactureDate": "2026-01-15",
  "warrantyPeriod": 24,
  "remarks": "정상품",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

**Error (404):** 시리얼을 찾을 수 없음

---

### 14.3 시리얼 번호 생성
```
POST /material-serials
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "serialNumber": "SN-2026-001",
  "materialId": 1,
  "locationId": 1,
  "manufactureDate": "2026-01-15",
  "warrantyPeriod": 24,
  "remarks": "정상품"
}
```

**Response (201):**
```json
{
  "id": 1,
  "serialNumber": "SN-2026-001",
  "status": "IN_STOCK",
  "createdAt": "2026-02-20T08:00:00.000Z"
}
```

---

### 14.4 시리얼 상태 변경
```
PATCH /material-serials/:id/status
```
🔒 **인증 필요**

**Request Body:**
```json
{
  "status": "IN_USE"
}
```

**Response (200):**
```json
{
  "id": 1,
  "status": "IN_USE",
  "updatedAt": "2026-02-20T09:00:00.000Z"
}
```

---

### 14.5 시리얼 삭제
```
DELETE /material-serials/:id
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "message": "삭제 성공"
}
```

**Error (400):** 삭제 불가 (사용중)

---

### 14.6 시리얼 이력 조회
```
GET /material-serials/:id/history
```
🔒 **인증 필요**

**Response (200):**
```json
[
  {
    "id": 1,
    "serialId": 1,
    "action": "CREATED",
    "fromStatus": null,
    "toStatus": "IN_STOCK",
    "locationId": 1,
    "locationCode": "A-01-01",
    "performedBy": 1,
    "performedAt": "2026-02-20T08:00:00.000Z",
    "remarks": "입고"
  },
  {
    "id": 2,
    "serialId": 1,
    "action": "STATUS_CHANGED",
    "fromStatus": "IN_STOCK",
    "toStatus": "IN_USE",
    "siteId": 1,
    "siteName": "강남 오피스텔 신축",
    "performedBy": 2,
    "performedAt": "2026-02-21T10:00:00.000Z",
    "remarks": "현장 출고"
  }
]
```

---

## 15. 대시보드 (Dashboard)

### 15.1 대시보드 전체 통계
```
GET /dashboard/stats
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "totalMaterials": 150,
  "totalStockValue": 500000000,
  "lowStockMaterials": 12,
  "expiringStockItems": 5,
  "pendingPurchaseOrders": 8,
  "activeSites": 15,
  "pendingQuotations": 3,
  "monthlyStockMovements": 245
}
```

---

### 15.2 재고 현황 차트 데이터
```
GET /dashboard/stock-chart
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "byCategory": [
    {
      "categoryName": "보안장비",
      "totalQuantity": 500,
      "totalValue": 75000000
    },
    {
      "categoryName": "네트워크장비",
      "totalQuantity": 300,
      "totalValue": 45000000
    }
  ],
  "byWarehouse": [
    {
      "warehouseName": "본사 창고",
      "totalQuantity": 600,
      "totalValue": 90000000
    },
    {
      "warehouseName": "지점 창고",
      "totalQuantity": 200,
      "totalValue": 30000000
    }
  ]
}
```

---

### 15.3 알림 목록
```
GET /dashboard/alerts
```
🔒 **인증 필요**

**Response (200):**
```json
[
  {
    "type": "LOW_STOCK",
    "severity": "HIGH",
    "message": "IP 카메라 재고 부족 (현재: 3, 안전재고: 10)",
    "materialId": 1,
    "materialCode": "MAT001",
    "materialName": "IP 카메라",
    "createdAt": "2026-02-20T08:00:00.000Z"
  },
  {
    "type": "EXPIRING_STOCK",
    "severity": "MEDIUM",
    "message": "LOT-2026-001 유효기한 임박 (23일 남음)",
    "stockId": 5,
    "lotNumber": "LOT-2026-001",
    "expiryDate": "2026-03-15",
    "createdAt": "2026-02-20T08:00:00.000Z"
  }
]
```

**알림 유형:**
- `LOW_STOCK`: 재고 부족
- `EXPIRING_STOCK`: 유효기한 임박
- `PENDING_APPROVAL`: 승인 대기

**심각도:**
- `HIGH`: 높음
- `MEDIUM`: 중간
- `LOW`: 낮음

---

### 15.4 최근 활동 내역
```
GET /dashboard/recent-activities?days=7
```
🔒 **인증 필요**

**Query Parameters:**
- `days`: number (선택, 기본값: 7, 조회 일수)

**Response (200):**
```json
[
  {
    "id": 1,
    "activityType": "STOCK_MOVEMENT",
    "action": "PURCHASE_IN",
    "description": "IP 카메라 50개 입고",
    "userId": 1,
    "userName": "홍길동",
    "createdAt": "2026-02-20T08:00:00.000Z"
  },
  {
    "id": 2,
    "activityType": "PURCHASE_ORDER",
    "action": "APPROVED",
    "description": "발주 PO-2026-001 승인",
    "userId": 2,
    "userName": "김관리",
    "createdAt": "2026-02-20T09:00:00.000Z"
  }
]
```

---

## 16. 리포트 (Reports)

### 16.1 재고 이동 리포트
```
GET /reports/stock-movements?startDate=2026-02-01&endDate=2026-02-28
```
🔒 **인증 필요**

**Query Parameters:**
- `startDate`: string (필수, YYYY-MM-DD)
- `endDate`: string (필수, YYYY-MM-DD)

**Response (200):**
```json
{
  "period": {
    "startDate": "2026-02-01",
    "endDate": "2026-02-28"
  },
  "summary": {
    "totalMovements": 245,
    "purchaseIn": 80,
    "siteOut": 120,
    "returnIn": 25,
    "returnOut": 10,
    "transfer": 10
  },
  "byMaterial": [
    {
      "materialId": 1,
      "materialCode": "MAT001",
      "materialName": "IP 카메라",
      "purchaseIn": 50,
      "siteOut": 40,
      "returnIn": 2,
      "returnOut": 1,
      "transfer": 5,
      "netChange": 6
    }
  ]
}
```

---

### 16.2 발주 리포트
```
GET /reports/purchase-orders?startDate=2026-02-01&endDate=2026-02-28
```
🔒 **인증 필요**

**Query Parameters:**
- `startDate`: string (필수, YYYY-MM-DD)
- `endDate`: string (필수, YYYY-MM-DD)

**Response (200):**
```json
{
  "period": {
    "startDate": "2026-02-01",
    "endDate": "2026-02-28"
  },
  "summary": {
    "totalOrders": 25,
    "totalAmount": 250000000,
    "approvedOrders": 20,
    "pendingOrders": 3,
    "cancelledOrders": 2
  },
  "bySupplier": [
    {
      "supplierId": 1,
      "supplierName": "한국전자",
      "orderCount": 10,
      "totalAmount": 100000000,
      "averageAmount": 10000000
    }
  ]
}
```

---

### 16.3 자재 사용 리포트
```
GET /reports/material-usage?startDate=2026-02-01&endDate=2026-02-28
```
🔒 **인증 필요**

**Query Parameters:**
- `startDate`: string (필수, YYYY-MM-DD)
- `endDate`: string (필수, YYYY-MM-DD)

**Response (200):**
```json
{
  "period": {
    "startDate": "2026-02-01",
    "endDate": "2026-02-28"
  },
  "materials": [
    {
      "materialId": 1,
      "materialCode": "MAT001",
      "materialName": "IP 카메라",
      "totalUsed": 120,
      "totalValue": 18000000,
      "bySite": [
        {
          "siteId": 1,
          "siteName": "강남 오피스텔 신축",
          "quantity": 80,
          "value": 12000000
        },
        {
          "siteId": 2,
          "siteName": "판교 사옥 신축",
          "quantity": 40,
          "value": 6000000
        }
      ]
    }
  ]
}
```

---

### 16.4 공급사별 발주 리포트
```
GET /reports/suppliers?startDate=2026-02-01&endDate=2026-02-28
```
🔒 **인증 필요**

**Query Parameters:**
- `startDate`: string (필수, YYYY-MM-DD)
- `endDate`: string (필수, YYYY-MM-DD)

**Response (200):**
```json
{
  "period": {
    "startDate": "2026-02-01",
    "endDate": "2026-02-28"
  },
  "suppliers": [
    {
      "supplierId": 1,
      "supplierCode": "SUP001",
      "supplierName": "한국전자",
      "orderCount": 10,
      "totalAmount": 100000000,
      "averageAmount": 10000000,
      "onTimeDeliveryRate": 95,
      "topMaterials": [
        {
          "materialId": 1,
          "materialCode": "MAT001",
          "materialName": "IP 카메라",
          "quantity": 500,
          "amount": 75000000
        }
      ]
    }
  ]
}
```

---

### 16.5 재고 회전율 리포트
```
GET /reports/inventory-turnover
```
🔒 **인증 필요**

**Response (200):**
```json
{
  "period": "최근 12개월",
  "materials": [
    {
      "materialId": 1,
      "materialCode": "MAT001",
      "materialName": "IP 카메라",
      "averageStock": 100,
      "totalUsed": 1200,
      "turnoverRate": 12,
      "turnoverDays": 30,
      "status": "GOOD"
    },
    {
      "materialId": 2,
      "materialCode": "MAT002",
      "materialName": "케이블",
      "averageStock": 500,
      "totalUsed": 600,
      "turnoverRate": 1.2,
      "turnoverDays": 304,
      "status": "SLOW"
    }
  ]
}
```

**회전율 상태:**
- `GOOD`: 양호 (회전율 > 6)
- `NORMAL`: 보통 (회전율 3-6)
- `SLOW`: 느림 (회전율 < 3)

---

## 부록

### A. 공통 Enum 값

#### 사용자 역할 (User Role)
- `ADMIN`: 관리자
- `MANAGER`: 매니저
- `USER`: 일반 사용자

#### 현장 상태 (Site Status)
- `PLANNING`: 계획
- `IN_PROGRESS`: 진행중
- `COMPLETED`: 완료
- `SUSPENDED`: 중단

#### 재고 이동 유형 (Movement Type)
- `PURCHASE_IN`: 구매 입고
- `SITE_OUT`: 현장 출고
- `RETURN_IN`: 현장 반품 입고
- `RETURN_OUT`: 공급사 반품 출고
- `TRANSFER`: 창고 간 이동

#### 승인 상태 (Approval Status)
- `PENDING`: 대기
- `APPROVED`: 승인
- `REJECTED`: 거절
- `CANCELLED`: 취소

#### 견적 상태 (Quotation Status)
- `DRAFT`: 작성중
- `SUBMITTED`: 제출
- `APPROVED`: 승인
- `REJECTED`: 거절
- `EXPIRED`: 만료

#### 발주 상태 (Purchase Order Status)
- `DRAFT`: 작성중
- `SUBMITTED`: 제출
- `APPROVED`: 승인
- `ORDERED`: 발주완료
- `RECEIVED`: 입고완료
- `CANCELLED`: 취소

#### 시리얼 상태 (Serial Status)
- `IN_STOCK`: 재고
- `IN_USE`: 사용중
- `DEFECTIVE`: 불량
- `RETURNED`: 반품

---

### B. 에러 코드

| 상태 코드 | 설명 |
|---------|------|
| 200 | 성공 |
| 201 | 생성 성공 |
| 400 | 잘못된 요청 |
| 401 | 인증 실패 |
| 403 | 권한 없음 |
| 404 | 리소스를 찾을 수 없음 |
| 409 | 충돌 (중복 데이터) |
| 500 | 서버 오류 |

---

### C. 파일 업로드 제한

| 필드 | 허용 형식 | 최대 크기 |
|-----|---------|---------|
| 사용자 이미지 | JPEG, PNG, GIF | 5MB |
| 자재 이미지 | JPEG, PNG, GIF | 5MB |

업로드된 파일은 `/uploads/{category}/{filename}` 경로에 저장됩니다.

---

### D. 날짜/시간 형식

- **날짜**: `YYYY-MM-DD` (예: 2026-02-20)
- **날짜/시간**: ISO 8601 형식 (예: 2026-02-20T08:00:00.000Z)
- **타임존**: UTC

---

### E. 페이징 응답 형식

목록 조회 API는 다음 형식으로 응답합니다:

```json
{
  "data": [...],
  "total": 100,
  "page": 1,
  "limit": 10,
  "totalPages": 10
}
```

---

## 변경 이력

| 버전 | 날짜 | 변경 내용 |
|-----|------|---------|
| 1.0 | 2026-02-20 | 초기 버전 작성 |

---

**문서 작성일**: 2026-02-20  
**API 버전**: 1.0  
**문의**: dev@example.com
