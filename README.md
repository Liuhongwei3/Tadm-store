> A Vue.js project

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```
## Vue实现电商网站项目@Tadm
`vue` +`Bootstrap`+`JQuery`+ `vue-router` + `vuex`+`sessionStorage` 实现电商网站

## Tools(有关环境配置请自行百度安装调试)
Webstorm/Vscode(个人极力推荐)
Google chrome
vue-cli
npm
git-bash

## 功能实现
1. 登录页面、网站首页、购物车页、商品详情页
2. 可按颜色、品牌对商品进行筛选，单击选中，再次点击取消
3. 根据价格进行升序降序、销量降序排列
4. 商品列表显示图片、名称、销量、颜色、单价
5. 实时显示购物车数量（商品类别数）
6. 购物车页面实现商品总价、总数进行结算，优惠券打折等操作

## 数据存储 & 数据处理

App.vue 获取相应数据并对sessionStoage进行处理
```javascript
export default {
    name: 'App',
    data(){
        return { 
            user: this.$store.state.username
        }
    },
    computed: {
        statu(){
            return this.$store.state.loginStatus;
        }
    },
    methods:{
        logout(){
            this.$store.state.username='';
            sessionStorage.clear();
        }
    }
    }
```

* `product.js`存放商品数据
```javascript
{
	id:1,
        name: '惠普(HP)暗影精灵4 Pro酷睿i7 15.6英寸游戏笔记本',
        brand:'惠普',
        image:'/src/images/HPAYJL4P.jpg',
        imagede1: '/src/images/HPAYJL4P1.jpg',
        imagede2: '/src/images/HPAYJL4P2.jpg',
        sales: 7000,
        cost: 11499,
        color: '黑色',
        number: 100003048788,
        weight: '4.05kg',
        city: '中国大陆',
        System: 'windows 10',
        capacity: '512GB SSD+1TB HDD',
        size: '15.6英寸',
        Graphics: 'RTX2070 8Gb 独显',
        CPU: 'i7-8750H 2.2GHz 六核',
        RAM: '16GB(2 x SO-DIMM)',
        Resolution: '1920×1080',
        Screen: 'IPS FHD'
},
```

* `window.sessionStorage`实现user数据存储与验证
```javascript
 Register(){
    if( !this.username || !this.password ){
        window.alert('账号或密码不能为空');
        return;
    }
    if(this.confirmPassword !== this.password){
        window.alert('密码不一致，请重新输入');
        this.password = '';
        this.confirmPassword = '';
    }
    else{
        window.sessionStorage.setItem('username', this.username);
        window.sessionStorage.setItem('password', this.password);
        this.register = false;
        window.sessionStorage.setItem('loginStatus', 'login');
        this.$store.commit('getUser', this.username);
        window.alert('注册成功，进入网站首页');
        this.$store.state.loginStatus=window.sessionStorage.getItem('loginStatus');
        this.$router.push('/');
    }
},
changeLogin(l, r){
    this.login = l;
    this.register = r;
},
toShop(){
    let username = window.sessionStorage.getItem('username');
    let password = window.sessionStorage.getItem('password');
    if( !this.username || !this.password){
        window.alert('账号或密码不能为空');
        return;
    }
    if(username === this.username && password === this.password){
        this.login = false;
        window.sessionStorage.setItem('loginStatus', 'login');
        this.$store.commit('getUser', this.username);
        window.alert('登录成功，进入网站首页');
        this.$store.state.loginStatus=window.sessionStorage.getItem('loginStatus');
        this.$router.push('/');
    }
    else{
        window.alert('账号或密码错误');
    }
},
```

**数据过滤与排序处理**
```javascript
import Product from './product.vue';
    export default {
        components: { Product },
        computed: {
            list(){//从Vuex获取商品列表信息
                return this.$store.state.productList;
            },
            brands(){
                return this.$store.getters.brands;
            },
            colors(){
                return this.$store.getters.colors;
            },
            filteredAndOrderedList(){
                let list = [...this.list];
                if(this.filterBrand !== ''){
                    list = list.filter(item => item.brand === this.filterBrand);
                }
                if(this.filterColor !== ''){
                    list = list.filter(item => item.color === this.filterColor);
                }
                if(this.order !== ''){
                    if(this.order === 'sales-desc'){ //销量排序
                        list = list.sort((a, b) => b.sales - a.sales);
                    }
                    else if(this.order === 'sales-asc'){
                        list = list.sort((a, b) => a.sales - b.sales);
                    }
                    else if(this.order === 'cost-desc'){ //价格排序
                        list = list.sort((a, b) => b.cost - a.cost);
                    }
                    else if(this.order === 'cost-asc'){
                        list = list.sort((a, b) => a.cost - b.cost);
                    }
                }
                return list;
            }
        },
        data(){
            return {
                filterBrand: '',
                filterColor: '',
                order: ''
            }
        },
        methods: {
            handleFilterDefault(){
                this.filterBrand = '';
                this.filterColor = '';
            },
            handleFilterBrand(brand){ //品牌筛选操作
                if (this.filterBrand === brand) {
                    this.filterBrand = '';//default
                }else{
                    this.filterBrand = brand;
                }
            },
            handleFilterColor(color){ //颜色筛选操作
                if (this.filterColor === color) {
                    this.filterColor = '';//default
                }else{
                    this.filterColor = color;
                }
            },
            handleOrderDefault(){
                this.order = '';
            },
            handleOrderSales(){ //销量状态转变
                if(this.order === 'sales-desc'){
                    this.order = 'sales-asc';
                }
                else{
                    this.order = 'sales-desc';
                }
            },
            handleOrderCost(){ //价格状态转变
                if(this.order === 'cost-desc'){
                    this.order = 'cost-asc';
                }
                else{
                    this.order = 'cost-desc';
                }
            },

        },
        mounted(){
            this.$store.dispatch('getProductList');
        }
    }
```

**实时显示应付总额与商品数**
```javascript
countAll(){
	let count = 0;
	this.cartList.forEach(item => {
		count += item.count;
	});
	return count;
},
costAll(){
	let cost = 0;
	this.cartList.forEach(item => {
		cost += this.productDictList[item.id].cost * item.count;
	});
	return cost;
}
```

**进行商品列表分页**
```javascript
<nav><!-- 实现JQuery分页组件 -->
    <ul class="pagination">
        <template v-for="page in Math.ceil(filteredAndOrderedList.length/pagesize)">
            <li v-on:click="PrePage()" id="prepage" v-if="page==1" class="disabled"><a href="#">上一页</a></li>
            <li v-if="page==1" class="active" v-on:click="NumPage(page, $event)"><a href="#">{{page}}</a></li>
            <li v-else v-on:click="NumPage(page, $event)"><a href="#">{{page}}</a></li>
            <li id="nextpage" v-on:click="NextPage()" v-if="page==Math.ceil(filteredAndOrderedList.length/pagesize)"><a href="#">下一页</a></li>
        </template>
    </ul>
</nav>
PrePage: function (event) {
    $(".pagination .active").prev().trigger("click");
},
NextPage: function (event) {
    $(".pagination .active").next().trigger("click");
},
NumPage: function (num, event) {
    if (this.curpage == num) {
        return;
    }
    this.curpage = num;
    $(".pagination li").removeClass("active");//移除active
    if (event.target.tagName.toUpperCase() == "LI") {//获取标签
        $(event.target).addClass("active");
    }
    else {
        $(event.target).parent().addClass("active");//A标签
    }
    if (this.curpage == 1) {
        $("#prepage").addClass("disabled");
    }
    else {
        $("#prepage").removeClass("disabled");
    }
    if (this.curpage == Math.ceil(this.filteredAndOrderedList.length / this.pagesize)) {
        $("#nextpage").addClass("disabled");
    }
    else {
        $("#nextpage").removeClass("disabled");
    }
}
```

**购物车数据处理以及结算处理**
```javascript
import product_data from '../product';
    
    export default {
        name: "cart",
        data(){
            return {
                promotion: 0,
                promotionCode: '',
                productList: product_data
            }
        },
        computed: {
            cartList(){
                return this.$store.state.cartList;
            },
            productDictList(){
                const dict = {};
                this.productList.forEach(item => {
                    dict[item.id] = item;
                });
                return dict;
            },
            countAll(){
                let count = 0;
                this.cartList.forEach(item => {
                    count += item.count;
                });
                return count;
            },
            costAll(){
                let cost = 0;
                this.cartList.forEach(item => {
                    cost += this.productDictList[item.id].cost * item.count;
                });
                return cost;
            }
        },
        methods: {
            handleOrder(){
                this.$store.dispatch('buy').then(() => {
                    window.alert('购买成功');
                })
            },
            handleCheckCode(){
                if(this.promotionCode === ''){
                    window.alert('请输入优惠码');
                    return;
                }
                if(this.promotionCode !== 'tadm'){
                    window.alert('优惠码输入错误');
                    return;
                }
                this.promotion = 666;
            },
            handleCount(index, count){
                if(count < 0 && this.cartList[index].count === 1) return;
                this.$store.commit('editCartCount', {
                    id: this.cartList[index].id,
                    count: count
                });
            },
            handleDelete(index){
                this.$store.commit('deleteCart', this.cartList[index].id);
            }
        }
    }

handleOrder(){
	this.$store.dispatch('buy').then(() => {
		window.alert('购买成功');
	})
},
```

## vue-router & vuex

**vue-router路由管理**`/src/views/`目录下的`vue`组件进行设置，`router-views`

**routers配置**

```javascript
Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'list',
      component: list
    },
    {
      path: '/productDetail/:id',
      name: 'productDetail',
      component: productDetail
    },
    {
      path: '/login/:loginStatus',
      name: 'login',
      component: login
    },
    {
      path: '/cart',
      name: 'cart',
      component: cart
    },
  ]
})
```

**vuex**状态管理，各组件共享数据在`state`中设置，`mutation`实现数据同步，`action`异步加载

```javascript
Vue.use(Vuex)

const store = new Vuex.Store({
    state: {
        productList: [],
        cartList: [],
        username: window.sessionStorage.getItem('username'),
        loginStatus: window.sessionStorage.getItem('loginStatus'),
    },
    getters: {
        brands: state => {
            //map返回一个新数组，原数组不会发生变化且适用于数组内所有元素使用同一种运算
            //Map转为Array使用扩展运算符... Set用于当前数组去重自动过滤
            const brands =[...new Set(state.productList.map(item => item.brand))];
            return brands;
        },
        colors: state => {
            const colors = [...new Set(state.productList.map(item => item.color))];
            return colors;
        }
    },
    mutations: {
        setProductList(state, data){
            state.productList = data;
        },
        addCart(state, id){
            const add = state.cartList.find(item => item.id === id);
            if(add){
                add.count++;
            }
            else{
                state.cartList.push({
                    id: id,
                    count: 1
                })
            }
        },
        //edit cart product count
        editCartCount(state, payload){
            const product = state.cartList.find(item => item.id === payload.id);
            product.count += payload.count;
        },
        //delete cart selected id product
        deleteCart(state, id){
            const index = state.cartList.findIndex(item => item.id === id);
            state.cartList.splice(index, 1)
        },
        //clear cart
        emptyCart(state){
            state.cartList = [];
        },
        getUser(state, username){
            state.username = username;
        },
        getLoginStatus(state, flag){
            state.loginStatus = flag;
        }
    },
    actions: {
        //异步请求product list，use setTimeout
        getProductList(context){
            setTimeout(() => {
                context.commit('setProductList', product_data)
            }, 500);
        },
        buy(context){
            //After use ajax ask for server do,then clear cart
            return new Promise(resolve => {
                setTimeout(() => {
                    context.commit('emptyCart');
                    resolve();
                }, 500);
            });
        },
    }
  })

  export default store
```

## About
Up to today(2019/6/17),the project will not update!

项目地址: [github](https://github.com/Liuhongwei3/Tadm-store) 
If the vue project can be helpful with you,please give me a star~  Thanks!

个人QQ `2873126657` Welcome,we can study more each other！