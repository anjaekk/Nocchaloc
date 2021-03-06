# ๐ต ๋น์ฐจ๋ก ๐ต

## ๐ ํ๋ก์ ํธ ๊ฐ์
- **์ด๊ธฐ์ธํ๋ถํฐ ๋ชจ๋ ์ง์  ์งํํ ์ค์ค๋ก ์ฌ์ดํธ ํด๋ก  ํ๋ก์ ํธ**
- **๊ธฐ ๊ฐ**: 2021.07.05.~2021.07.16.(12์ผ)
- **Team member**
  - Frontend: 2์ธ(๊น์์ข, ์ค์ง์)
  - Backend: 3์ธ(๋ฐ์ฑ์ค, ์์ฌ๊ฒฝ, ์ ์ํ)
  
## ๐ ํ๋ก์ ํธ ๋ฏธ๋ฆฌ๋ณด๊ธฐ      

![ezgif com-gif-maker](https://user-images.githubusercontent.com/74139727/126070910-dd2c6151-0245-436d-a2ab-12d2e0653079.gif)

## ๐ ํ๋ก์ ํธ ์งํ
<details>
<summary>Trello๋ฅผ ์ด์ฉํ Scrum๊ด๋ฆฌ</summary>
<div markdown="1">       

![](https://images.velog.io/images/anjaekk/post/3bcb6e9b-b876-4447-9fd4-4b36af8a4182/image.png)

</div>
</details>


- ๋งค์ผ ์ด์  ํ ์ผ, ์ค๋ ํ  ์ผ, ํน์ด์ฌํญ ๊ณต์ 
- ๋ฏธํ์  Agenda ๊ณต์  

## ๐ Database ์ค๊ณ

<details>
<summary>Database modeling</summary>
<div markdown="2">       

![](https://images.velog.io/images/anjaekk/post/47adf0ea-2c4f-48f1-b8cc-d6590b3cc62e/image.png)

</div>
</details>

## ๐ ๋ด๊ฐ ๋งก์ ํํธ

<details>
<summary>์ฅ๋ฐ๊ตฌ๋ CRUD ๊ตฌํ</summary>
<div markdown="2">

- ์ฅ๋ฐ๊ตฌ๋ ์ถ๊ฐ
```
'''(์๋ต)
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
'''(์๋ต)
```
  
- ์ฅ๋ฐ๊ตฌ๋ ๋ณด๊ธฐ
```
'''(์๋ต)
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
'''(์๋ต)
```

- ์ฅ๋ฐ๊ตฌ๋ ์์ 
```
'''(์๋ต)
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
'''(์๋ต)
```
  
- ์ฅ๋ฐ๊ตฌ๋ ์ญ์ 
```
'''(์๋ต)
    @authorization
    def delete(self, request):
        try:
            carts_id = request.GET.getlist('cart_id')

            for cart in carts_id:
                if not Cart.objects.filter(user=request.user, id=int(cart)).exists:
                    return JsonResponse({'message':'VALUE_ERROR'}, status=404)
                Cart.objects.get(user=request.user, id=int(cart)).delete()
'''(์๋ต)
```

  
</div>
</details>



<details>
<summary>์ ํ ๊ฒ์๊ธฐ๋ฅ ๊ตฌํ</summary>
<div markdown="2">

- ์ด๋ฆ๊ณผ ์ ํ ์ค๋ช์์ ๊ฒ์ ํ review์์ผ๋ก ์ ๋ ฌ

```
'''(์๋ต)
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
'''(์๋ต)
```

</div>
</details>


## ๐ ์ ์ฒด ๊ตฌํ ๊ธฐ๋ฅ
|User(ํ์๊ฐ์ ๋ฐ ๋ก๊ทธ์ธ)|Product(์ ํ ๋ฆฌ์คํธ ๋ฐ ๋ฆฌ๋ทฐ)|Order(์ฅ๋ฐ๊ตฌ๋ ๋ฐ ๊ฒฐ์ )|
|:-:|:-:|:-:|
|Bcrypt ์ํธํ|pagination|์ฅ๋ฐ๊ตฌ๋ CRUD|
|JWT Access Token ์ ์ก|์ ์ํ, ๊ฐ๊ฒฉ, ์กฐํ์, ๋ฆฌ๋ทฐ ๊ธฐ์ค ์ ๋ ฌ|ํฌ์ธํธ ๊ฒฐ์ |
|ํ์๊ฐ์ ์ ํจ์ฑ ๊ฒ์ฌ ๊ตฌํ|์ ํ ํ์์ ๋ณ filtering|
||์ ํ ๊ฒ์(์ด๋ฆ, ์ค๋ช)|
||๋ฆฌ๋ทฐ CRUD||

## ๐ API Documentation
https://documenter.getpostman.com/view/16450829/Tzm8EErD
