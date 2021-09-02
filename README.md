# 🍵 녹차록 🍵

## 📍 프로젝트 개요
- **초기세팅부터 모두 직접 진행한 오설록 사이트 클론 프로젝트**
- **기 간**: 2021.07.05.~2021.07.16.(12일)
- **Team member**
  - Frontend: 2인(김수종, 오지수)
  - Backend: 3인(박성준, 안재경, 전수현)
  
## 📍 프로젝트 미리보기      

![ezgif com-gif-maker](https://user-images.githubusercontent.com/74139727/126070910-dd2c6151-0245-436d-a2ab-12d2e0653079.gif)

## 📍 프로젝트 진행
<details>
<summary>Trello를 이용한 Scrum관리</summary>
<div markdown="1">       

![](https://images.velog.io/images/anjaekk/post/3bcb6e9b-b876-4447-9fd4-4b36af8a4182/image.png)

</div>
</details>


- 매일 어제 한 일, 오늘 할 일, 특이사항 공유
- 미팅전 Agenda 공유 

## 📍 Database 설계

<details>
<summary>Database modeling</summary>
<div markdown="2">       

![](https://images.velog.io/images/anjaekk/post/47adf0ea-2c4f-48f1-b8cc-d6590b3cc62e/image.png)

</div>
</details>

## 📍 내가 맡은 파트

<details>
<summary>장바구니 CRUD 구현</summary>
<div markdown="2">

- 장바구니 추가
```
'''(생략)
@authorization
    def post(self, request):
        try:
            data       = json.loads(request.body)
            product_id = data['product_id']
            option_id  = data['option_id']
            quantity   = data['quantity']

            if Product.objects.filter(id=product_id).exists() and Option.objects.filter(id=option_id).exists():
                add_cart, is_create = Cart.objects.get_or_create(
                    user       = request.user, 
                    product_id = product_id, 
                    option_id  = option_id,)
                add_cart.quantity += quantity
                add_cart.save()
'''(생략)
```
  
- 장바구니 보기
```
'''(생략)
    @authorization
    def get(self, request):
        try:
            if Cart.objects.filter(user=request.user).exists():
                carts = Cart.objects.filter(user=request.user)

                cart_list = [{
                    'product'    : cart.product.name,
                    'quantity'   : cart.quantity,
                    'option'     : cart.option.name,
                    'unit_price' : cart.product.price,
                    'price'      : cart.product.price * cart.quantity
                } for cart in carts]
'''(생략)
```

- 장바구니 수정
```
'''(생략)
    @authorization
    def patch(self, request):
        try:
            product_id = request.GET['product_id']
            option_id  = request.GET['option_id']  
            operation  = request.GET['operation']

            if Cart.objects.filter(user=request.user, product=product_id, option=option_id).exists():
                change_cart = Cart.objects.get(user=request.user, product=product_id, option=option_id)
                if operation == 'add':
                    change_cart.quantity += 1

                if operation == 'subtraction':
                    change_cart.quantity -= 1
                change_cart.save()
'''(생략)
```
  
- 장바구니 삭제
```
'''(생략)
    @authorization
    def delete(self, request):
        try:
            carts_id = request.GET.getlist('cart_id')

            for cart in carts_id:
                if not Cart.objects.filter(user=request.user, id=int(cart)).exists:
                    return JsonResponse({'message':'VALUE_ERROR'}, status=404)
                Cart.objects.get(user=request.user, id=int(cart)).delete()
'''(생략)
```

  
</div>
</details>



<details>
<summary>제품 검색기능 구현</summary>
<div markdown="2">

- 이름과 제품 설명에서 검색 후 review순으로 정렬

```
'''(생략)
def get(self, request):
        try:
            PAGE_SIZE   = 24
            limit       = int(request.GET.get('limit', PAGE_SIZE))
            offset      = int(request.GET.get('offset', 0))
            keyword     = request.GET.get('keyword', None)

            search_list = Product.objects.filter(Q(name__icontains=keyword) | Q(description__icontains=keyword)) \
            .annotate(review_count=Count('review')).order_by('-review_count')[offset:limit]
            context     = [{
                'id'             :search.id,
                'name'           :search.name,
                'price'          :search.price,
                'main_image_url' :search.main_image_url,
                'hover_image_url':search.hover_image_url
            } for search in search_list]
'''(생략)
```

</div>
</details>


## 📍 전체 구현 기능
|User(회원가입 및 로그인)|Product(제품 리스트 및 리뷰)|Order(장바구니 및 결제)|
|:-:|:-:|:-:|
|Bcrypt 암호화|pagination|장바구니 CRUD|
|JWT Access Token 전송|신상품, 가격, 조회수, 리뷰 기준 정렬|포인트 결제|
|회원가입 유효성 검사 구현|제품 타입에 별 filtering|
||제품 검색(이름, 설명)|
||리뷰 CRUD||

## 📍 API Documentation
https://documenter.getpostman.com/view/16450829/Tzm8EErD
