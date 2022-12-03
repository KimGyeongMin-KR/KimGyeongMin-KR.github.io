---
title : Serializer 객체 접근 & 게시글에 포함 된 여러장의 이미지를 serializer로 저장하기
use_math : true
tags : python django drf django-rest-framework
category : [Programming Django]

---

Serializer 객체 접근 & 게시글에 포함 된 여러장의 이미지를 serializer로 저장
===

## Serializer 객체 접근할 수 있는 3가지

### initial_data
- 유효성 검사를 하기 전에 필드에 접근할 수 있습니다.
```python
serializer = serializer_class(data = request.data)
serializer.initial_data['필드'] = 값 or 필드 값을 이용할 수 있음
```

### validated_data
- 유효성 검사를 통과한 필드에 접근 할 수 있다.
```python
serializer = serializer_class(data = request.data)
if serializer.is_valid():
    serializer.validated_data['필드'] = 값
    serializer.save()
```

### data
- 유효성 검사를 통과하고 save된 필드에 접근할 수 있다.
```python
serializer = serializer_class(data=request.data)
if serializer.is_valid():
    serializer.save()
    serializer.data['필드']
```

## 게시글에 포함 된 여러장의 이미지 serializer로 저장하기


### models.py
```python

class Goods(models.Model):
    class Meta:
        db_table = 'Goods'

    seller = models.ForeignKey(User, on_delete=models.CASCADE, related_name='sell_goods')
    title = models.CharField(max_length=256)
    content = models.TextField()



class GoodsImage(models.Model):
    goods = models.ForeignKey(Goods, on_delete=models.CASCADE)
    image = models.ImageField(upload_to='goods/')

```
예시로 상품 모델이 있고 상품 이미지 모델을 만들었다.
상품에는 여러개의 이미지를 포함시키기 위해 상품이미지 모델을 분리 시켜서 상품을 참조하도록 하였다.


### serializers.py
```python


class GoodsImageSerializer(serializers.ModelSerializer):
    class Meta:
        model = GoodsImage
        fields = ['image',]


class GoodsDetailSerializer(serializers.ModelSerializer):
    seller = serializers.SerializerMethodField()
    goodsimage_set = GoodsImageSerializer(many=True, read_only = True)

    # images = serializers.SerializerMethodField()
    # def get_images(self, obj):
        # return GoodsImageSerializer(obj.goodsimage_set.all(), many = True).data

    def get_seller(self, obj):
        return obj.seller.username

    class Meta:
         model = Goods
         fields = '__all__'
```

상품 이미지 모델에서 상품을 참조하기 때문에 상품에서 역참조 시 related_name이 설정되어 있지 않다면 모델_set 필드로 사용하면 되고, 상품 이미지 시리얼 라이저를 통하여 many=True, read_only=True값을 주어서 이미지의 주소 값을 가져올 수 있다. 

(주석처럼 SerializerMethodField를 사용해 주어도 된다.)


### views.py
```python

class GoodsView(ModelViewSet):

    queryset = Goods.objects.prefetch_related('goodsimage_set').select_related('seller').all()
    serializer_class = GoodsDetailSerializer
    permission_classes = [IsAuthorOrReadOnly,]

    def get_permissions(self):
        if self.action == 'create':
            return [permissions.IsAuthenticated(),]
        return super(GoodsView, self).get_permissions()

    
    def create(self, request, *args, **kwargs):

        serializer = self.get_serializer(data=request.data)

        images = serializer.initial_data.pop('goodsimage_set')  [1]
        images = [{'image' : image} for image in images]        [2]

        serializer.is_valid(raise_exception=True)
        obj = self.perform_create(serializer)               [3]

        image_serializer = GoodsImageSerializer(data = images, many = True) [4]
        image_serializer.is_valid(raise_exception=True) [4]
        image_serializer.save(goods = obj)  [5]

        headers = self.get_success_headers(serializer.data)
        return Response(serializer.data, status=status.HTTP_201_CREATED, headers=headers)


    def perform_create(self, serializer):
        return serializer.save(seller_id = self.request.user.id) # save()메서드는 생성된 object를 반환해 주기 때문에 오버라이딩하였다
```
모델 뷰셋을 사용하였는데 여기에서 중요한 것만 짚겠다.

post요청이 왔을 때 create메서드를 오바라이딩 하였다
- [1] 유효성 검사를 통하기 전 데이터(initial_data)에서 이미지 필드의 값들을 빼온다
- [2] 1번에서 빼온 이미지 데이터를 {'필드값' : 이미지값} serializer양식에 맞춰준다
- [3] 글 양식에서 이미지를 제외한 데이터를 serializer를 통해서 저장하고 객체를 가져온다.
- [4] 2번에서 양식을 맞춰준 이미지 데이터들을 시리얼 라이저에 넣어 유효성 검사를 한다.
- [5] 어떤 상품을 바라보는지 3번에서의 값을 저장시켜준다.


~~request.data에서 빼와서 사용을 해도 되지만 serializer에서 접근을 하고 싶은 마음에 검색하여 사용했다.~~ 

serializer클래서 내부에서 가공되지 않은 상태의 데이터를 사용하기 위해서 initial_data를 사용하는 것으로 판단을 하였다. context에 데이터를 넘겨주어도 되지만 여기서는 굳이 그렇게 안 하고 view에서 처리하였다.