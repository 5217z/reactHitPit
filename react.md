# react踩坑指南之dva

***
#### dva组成 

+ 可以分为主要的三个部分，***models***、***services*** 和 ***views***。其中，views负责页面上的展示，services里面主要写一些请求后台接口的方法；models是其中最重要的概念，这里存放了各种数据，并对数据进行相应的交互。


#### models 

+ ***存放数据和更新数据***
  ```
  import { Effect, ImmerReducer, Reducer, Subscription } from 'umi';
import api from '../http/home'
export interface IndexModelState {
    name: string;
    num: number;
    playlist: any[];
    ImgUrl: string
}

export interface IndexModelType {
    namespace: 'index';
    state: IndexModelState;
    effects: {
        Getbanner: Effect;

    };
    reducers: {
        save: Reducer<IndexModelState>;
        addnum: Reducer<IndexModelState>;
        changeImg: Reducer<IndexModelState>;
        // 启用 immer 之后
        // save: ImmerReducer<IndexModelState>;
    };
    subscriptions: { setup: Subscription };
}

const IndexModel: IndexModelType = {
    namespace: 'index',  // 命名空间  必须全局唯一

    state: {  // 存放数据 类似于vuex的state
        name: 'asdasdjkasd',
        ImgUrl: '',
        playlist: [],
        num: 1,
    },

    effects: {  //异步发送请求  // 类似于vuex的action
        *Getbanner({ payload }, { call, put }) {  // Getbanner 发送请求的方法名; call：发送请求;put ：更新数据 

            const result = yield call(api.getBanner)  // api:导入的axios请求的对象名，getBanner请求的具体方法名
            let res = result.data.banners
            console.log(22);
            yield put({ type: "save", payload: { playlist: res, ImgUrl: res[0].imageUrl } }); // yield表示同步调用，这个是generator提供的功能  type :reducers的方法名

        },


    },
    reducers: {  // 同步调用  用于存放能够改变view的action
        save(state, action) {

            console.log(111);

            return {
                ...state,
                ...action.payload,
            };
        },


        addnum(state, action) {
            console.log(111);

            return {
                ...state,
                ...action.payload,
            };
        },

        changeImg(state, action) {

            return {
                ...state,
                ...action.payload,
            };

        }

        // 启用 immer 之后
        // save(state, action) {
        //   state.name = action.payload;
        // },
    },
    subscriptions: {  //订阅监听，比如监听路由，进入页面如何如何，就可以在这里处理。相当于原生React中的componentWillMount方法。就比如上述代码，监听/project路由，进入该路由页面后，将发起getAllProjects aciton，获取页面数据。
        setup({ dispatch, history }) {
            return history.listen(({ pathname }) => {
                if (pathname === '/') {
                    dispatch({
                        type: 'qweW',
                    })
                }
            });
        }
    }
};

export default IndexModel;
  ```
#### services 

+ 发送请求的页面 
+ ***就是一般发送请求方式***
#### views 

+ views主要是页面的展示
+ 在views调用models 的方法 ***类组件使用方式***
```
import React from 'react'
import Getbanner from '../../http/home'
import { Carousel } from 'antd';
import banstyle from '../../style/home/home.module.less'
import { IndexModelState, ConnectProps, Loading, connect, useDispatch } from 'umi';
class Banner extends React.Component<any, any> {

    constructor(props: any) {
        super(props)
        this.state = {
            banner: [],
            dotPosition: "top",
            Imgurl: null,
        }
        this.props.dispatch({   //调用models的方法 相当于vue的this.$store.commit()

            type: "index/Getbanner",   //  models的命名空间/调用的方法名
        })


    }

    changeImg(from: any, to: any) {
        console.log(this.props.ImgUrl);
        this.props.dispatch({
            type: "index/changeImg",
            payload: {
                ImgUrl: this.props.playlist[to].imageUrl  // 调用是传递的参数
            }


        })
    }

    render() {
        return (
            <div style={{ position: "relative", width: "100vw" }} className={banstyle.banbg}>
                <div className={banstyle.banImgbg}>
                    <div style={{ backgroundImage: `url(${this.props.ImgUrl})` }} ></div>
                </div>
                <div style={{ width: 730 }} className={banstyle.banner}>
                    <Carousel effect="fade" autoplay={true} beforeChange={this.changeImg.bind(this)} style={{ borderRadius: 15 }}>
                        {
                            this.props.playlist.map((v: any, index: number) => {
                                return (
                                    <div className={banstyle.contentStyle} key={v.targetId}>

                                        <img src={v.imageUrl} alt="" />
                                    </div>
                                )
                            })
                        }
                    </Carousel>
                </div>

            </div>
        )
    }







}

const mapStateToProps = (state: any) => {  // state 和 models的 映射
    const {
        name,
        playlist,
        ImgUrl
    } = state.index;   // 你需要使用的数据
    return {
        name,
        playlist,
        ImgUrl       
    };
};
export default connect(mapStateToProps)(Banner) //类组件名  
  ```

 + 在views调用models 的方法 ***函数组件使用方式***

```
import React, { FC } from 'react';
import { IndexModelState, ConnectProps, Loading, connect, useDispatch } from 'umi';


interface PageProps extends ConnectProps {
  index: IndexModelState;
  loading: boolean;
}
const IndexPage: FC<PageProps> = ({ index }) => {  // idnex 代表命名空间的名字
  const { name, num } = index;  // 可以直接解构 models 的数据

  let dispatch = useDispatch() // 与类组件不同 先获取 dispatch; useDispatch() 必须使用从umi中解构 
  let add = (num: any) => dispatch({ type: "index/addnum", payload: { num: num + 1 } });   // 调用 models 的方法



  return (<div >

    <div>
      Hello {name}
    </div>
    <button onClick={() => { add(num) }}>
      asdasdasd
      {num}
    </button>
  </div>


  )


};





export default connect(({ index, loading }: { index: IndexModelState; loading: Loading }) => ({
  index,  // 命名空间的名字  必须与 models命名空间一样
  loading: loading.models.index,
}))(IndexPage);  // 函数组件



```



## 