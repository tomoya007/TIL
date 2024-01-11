# 起きている問題
新規登録を実行した時に、なぜかログイン状態が保持されず、ログイン画面まで戻ってしまう。  
（ログアウト状態では、ログイン画面に自動的に遷移するように設定してある）


デベロッパーコンソールは下記↓
```
SignupComponent.js:31 Token saved: undefined
```

ネットワークタブの```users/```のResponseは下記。  
見ての通り、レスポンスボディに```token```フィールドが含まれていません。↓
```
{"id":5,"username":"abcde","email":"abcde@gmail.com"}
```

また、Djangoのログに ```Generated token for user {username}: {token.key}``` の出力が確認できません。

# 原因考察
```views.py```の```signup_view```には、下記のように設定してあります。
```python
logger.debug(f"Generated token for user {username}: {token.key}")
```

しかしDjangoのログに ```Generated token for user {username}: {token.key}``` の出力が確認できないことから、Djangoの ```signup_view``` からのレスポンスデータにトークンが含まれていないことにあると考えられます。  

これは、トークンの生成またはレスポンスデータへの含め方に問題がある可能性があります。

### Djangoシェルでのトークン生成テスト
Djangoのシェルを使用してトークン生成を手動でテストしてみます。
```
docker exec -it mangabot_backend_1 python manage.py shell
```

```python
from django.contrib.auth import get_user_model
from rest_framework.authtoken.models import Token

user = get_user_model().objects.get(username='your_username')
token, created = Token.objects.get_or_create(user=user)
print(token.key)
```
これらのステップを踏んだ結果、正常にトークンが生成されたので、Djangoのトークン生成自体には問題がありませんでした。  

トークンがシェルで正常に生成されているにもかかわらず、APIを通じてフロントエンドにトークンが渡らない場合、問題は ```signup_view``` 関数またはその周辺のコード（例: シリアライザー、ビューセット、ルーティングなど）にある可能性があります。

# 解決
**結論：** シリアライザの定義が原因だった。

下記のように、```CustomUserViewSet``` が ```CustomUserSerializer``` を使用している場合、このシリアライザの定義がレスポンスデータの形式に直接影響を与えます。

```views.py
class CustomUserViewSet(viewsets.ModelViewSet):
    queryset = get_user_model().objects.all()
    serializer_class = CustomUserSerializer
```


 ```CustomUserSerializer```を確認していきます。（元々のコード）↓


```serializers.py
class CustomUserSerializer(serializers.ModelSerializer):
    class Meta:
        model = get_user_model()
        fields = ('id', 'username', 'email', 'password')
        extra_kwargs = {'password': {'write_only': True}}

```

上記の ```CustomUserSerializer``` は ```id, username, email, password``` フィールドを持つように定義されています。  

しかし、このシリアライザには ```token``` フィールドが含まれていないため、新規ユーザー登録のレスポンスにトークンが含まれない原因となっています。  

トークンをレスポンスに含めるには、```CustomUserSerializer``` にトークンフィールドを追加する必要があります。  
以下のように変更します。

```python
class CustomUserSerializer(serializers.ModelSerializer):
    token = serializers.SerializerMethodField()

    class Meta:
        model = get_user_model()
        fields = ('id', 'username', 'email', 'password', 'token')
        extra_kwargs = {'password': {'write_only': True}}

    def get_token(self, obj):
        token, created = Token.objects.get_or_create(user=obj)
        return token.key
```

無事レスポンスデータにトークンが含まれるようになり、新規登録ができました！
