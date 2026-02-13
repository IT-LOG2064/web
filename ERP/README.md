# 그룹웨어 ERP 시스템

## 프로젝트 개요
견적, 재고, 발주, 입고, 출고, AS, 자재정보 관리를 위한 통합 ERP 시스템

## Agent 구성

이 프로젝트는 4개의 Agent로 나누어 개발합니다:

1. **Agent 1: UI 컴포넌트 개발자** - 공통 컴포넌트 및 레이아웃
2. **Agent 2: 화면 구현 개발자** - 모든 화면 구현
3. **Agent 3: API/Database 설계자** - API 및 DB 설계
4. **Agent 4: 비즈니스 로직 개발자** - 모든 비즈니스 로직 구현

자세한 내용은 `.kiro/agents/README.md` 참조

## 시작하기

### 1. Agent 역할 확인
```bash
# Agent별 상세 가이드 확인
.kiro/agents/frontend-ui-components.md    # Agent 1
.kiro/agents/frontend-screens.md          # Agent 2
.kiro/agents/backend-api-database.md      # Agent 3
.kiro/agents/backend-business-logic.md    # Agent 4
```

### 2. 화면정의서 확인
- `그룹웨어 ERP 화면정의서.md` - 전체 화면 개요
- `견적 관리_화면정의서.md` - 견적 관리 상세
- `발주관리_화면정의서.md` - 발주 관리 상세
- `입고관리_화면정의서.md` - 입고 관리 상세
- `출고관리_화면정의서.md` - 출고 관리 상세
- `AS관리_화면정의서.md` - AS 관리 상세
- `자재정보관리_화면정의서.md` - 자재정보 관리 상세

### 3. HTML 목업 참조
- `dashboard.html` - 대시보드
- `contract.html` - 견적 관리
- `order.html` - 발주 관리
- `receiving.html` - 입고 관리
- `shipment.html` - 출고 관리
- `as.html` - AS 관리
- `materials.html` - 자재정보 관리
- `inventory.html` - 재고 관리

## 개발 우선순위

### Phase 1: 기반 구축
- Agent 3: DB 스키마 설계 및 마이그레이션
- Agent 3: API 엔드포인트 정의
- Agent 1: 공통 컴포넌트 개발

### Phase 2: 핵심 기능
- Agent 4 + Agent 2: 견적 관리
- Agent 4 + Agent 2: 발주 관리
- Agent 4 + Agent 2: 출고 관리

### Phase 3: 지원 기능
- Agent 4 + Agent 2: 입고 관리
- Agent 4 + Agent 2: AS 관리
- Agent 4 + Agent 2: 자재정보 관리

### Phase 4: 부가 기능
- Agent 4 + Agent 2: 재고 관리
- Agent 4 + Agent 2: 대시보드

## 주요 비즈니스 규칙

### ⭐⭐⭐ 필수 검증 사항
1. **견적 수정 시 수정 사유 필수**
2. **입고 검수 시 검수 수량 = 입고 수량**
3. **출고 시 재고 차감 정확성**
4. **시리얼 선택 개수 = 출고 수량**
5. **자재 삭제 조건 검증**

## 참조 데이터
- `materials-master.json` - 자재 마스터 샘플 데이터

## 문의
각 Agent별 가이드 문서를 참조하세요.
