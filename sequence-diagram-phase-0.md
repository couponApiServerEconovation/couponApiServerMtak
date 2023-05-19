# 쿠폰 등록
---
```mermaid
sequenceDiagram
    actor user
    actor cl as client
    participant ctr as controller
    participant s as service
    participant db
    
    user->>ctr : data: userId + couponNumber
    ctr->>s: registerCoupon(userId, couopnNumber)
    s->>db: getCouponInfo(couponNumber)
    db->>s: CouponInfo
    opt authentication needed
    Note over s, db: IsUserValidForCouponAuthenticationNeeded(userId)
    opt invalid
    s-->ctr: failure
    ctr-->user: failure
    end
    end
    Note over s, db: IsCouponRegistable(CouponInfo)
    alt True
         s-->ctr: failure
         ctr-->user: failure
    end
    s ->> db: update coupon list(userId, nowDate,couponNumber) 
    s-->> ctr: success 
    ctr-->> user: success
```

> [CouponInfo]  
> couponNumber
> 인증이 필요한가 authenticationNeeded
> couponStocks
> couponDueDate
> couponType

## IsUserValidForCouponAuthenticationNeeded(userId, couponType)
* couponType == 개인 지정 쿠폰
  * userId를 위한 쿠폰이 있나?
* couponType == 단체 지정 쿠폰
  * userId가 화이트 리스트에 있나?
```mermaid
sequenceDiagram
participant s as service
participant db
alt couponType == personalDesignationCoupon
s->> db: personalDesignationCoupon.findbyUserId()
else couponType == groupDesignationCoupon
s->>db: groupDesignationCoupon.findUserId()
end
db->>s: True/False
```

## IsCouponRegistrable(couponStocks, couponDueDate)
* 수량 제한이 있나?
  * 재고가 있나?
* 만기일이 있나?
  * 만기일 넘었나?
```mermaid
sequenceDiagram
participant s as service
alt couponStocks < 1 or Now() > couponDueDate
s->>s: False
end
s->>s: True

```
# 쿠폰 사용
```mermaid
sequenceDiagram
actor user
actor mk as market
participant ctr as controller
participant s as service
participant db

mk->>ctr: userId, couponNumber
ctr->>s: useCoupon(userId, couponNumber)
Note over s, db: isCouponUsable(userId, couponNumber)
alt usuable:
          s->>db: update coupon list, coupon use
          s-->>ctr: permitted
          ctr-->>mk: permitted
          mk-->>user: rejected
else unusable
s-->>ctr: rejected
ctr-->>mk: rejected
mk-->>user: rejected
end
```

## isCouponUsable(couponNumber, userId)
* 존재하는 쿠폰인가?
* 쿠폰 주인이 나인가?
* 유효 기간이 지났는가?
```mermaid
sequenceDiagram
participant s as service
participant sv as couponValidationService
participant db

s->>db: findCouponDueDateAndStockAndOwnerById(couponNumber)
db->>s: couponDueDate, couponStockCnt , ownerId || NULL
alt not NULL
  s->>sv: isCouponNotOutdated(couponDueDate) || isCouponOutOfStock(couponStockCnt) || ownerId != userId
  sv->>s: True || False
end

```

### CouponInfo
* couponDueDate : DATE
* couponStockCnt : int
* CouponNumber : int
* couponType : {SINGULAR_NUM_AUTH_NEEDED, PLURAL_NUM_AUTH_UNNECESSARY, 
  SINGULAR_NUM_REDUNDANT_RECEIPTABLE, PLURAL_NUM_NONREDUNDANT_RECEIPTABLE}
* couponUserWhiteList : 

# 쿠폰 발급
```mermaid
sequenceDiagram
actor cl as client
participant ctr as controller
participant s as service
participant db
cl->>ctr: CouponInfo
ctr->>s: issueCoupon(CouponInfo)
Note over s: isCouponValid(CouponInfo)
alt valid:
s->>db: update coupon list
s-->>ctr: issuedCouponInfo
ctr-->>cl: issuedCouponInf
else invalid:
ctr-->>cl: not issued
end
```

## isCouponValid(CouponInfo)
* 필수 입력이 다 들어왔는기?
  * 쿠폰 정책 이름
  * 쿠폰 할인 구분
  * 다운로드 가능 기간
  * 사용할 수 있는 기간
  * 중복 가능 여부
  * 수량 제한 여부
  * 고객 인증 필요 여부
  * 쿠폰 이미지

```mermaid
sequenceDiagram
participant s 
participant sv as couponValidationService

s->>sv: isCouponValid(CouponInfo)
sv->> s: True || False
```

## 고객사 리포팅 (보류)
```mermaid
sequenceDiagram
actor cl as client
participant ctr as controller
participant s as service
participant db
```

