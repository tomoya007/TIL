```npm install```を実行した際、エラーが出た。

```
% npm install

npm ERR! While resolving: react-scripts@5.0.1
npm ERR! Found: typescript@5.2.2
npm ERR! node_modules/typescript
npm ERR!   typescript@"^5.2.2" from the root project
npm ERR!   peer typescript@">= 2.7" from fork-ts-checker-webpack-plugin@6.5.3
npm ERR!   node_modules/fork-ts-checker-webpack-plugin
npm ERR!     fork-ts-checker-webpack-plugin@"^6.5.0" from react-dev-utils@12.0.1
npm ERR!     node_modules/react-dev-utils
npm ERR!       react-dev-utils@"^12.0.1" from react-scripts@5.0.1
npm ERR!       node_modules/react-scripts
npm ERR!         react-scripts@"5.0.1" from the root project
...
npm ERR! Could not resolve dependency:
npm ERR! peerOptional typescript@"^3.2.1 || ^4" from react-scripts@5.0.1
npm ERR! node_modules/react-scripts
npm ERR!   react-scripts@"5.0.1" from the root project
```
エラー文によると、プロジェクトの依存関係にあるバージョン ```react-scripts``` と ```typescript``` の競合が起きている。

具体的には、```react-scripts@5.0.1```は```typescript```のバージョン```^3.2.1``` または ```^```4を必要としているが、プロジェクトが使用しているtypescriptのバージョンは```5.2.2```であるとのこと。

```frontend/package.json
"react-scripts": "5.0.1",
"typescript": "^5.2.2",
```

### 解決策
**package.json の依存関係を更新する:**  
1.```typescript``` のバージョンを ```react-scripts``` に合わせるために更新する。    
```react-scripts@5.0.1``` は ```typescript@^3.2.1 || ^4``` を要求しているので、```typescript``` のバージョンを ```^4.9.5``` などに更新してみる。  

2.```package.json``` を更新した後、ローカル環境で ```npm install``` を再度実行する。  
→無事に実行できた！
