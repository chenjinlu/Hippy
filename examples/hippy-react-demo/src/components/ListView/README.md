### List组件

1. **简介** 

    List组件提供垂直列表功能，尤其适合大量条目的数据渲染。Hippy引擎针对内存、流畅度做了深度优化，
    在针对内存和流畅度的评测中均达到甚至超过了Native的水平。这个Demo展示了一个具有分页加载能力的feed流列表。

2. **效果截图**

![1](http://res.imtt.qq.com/hippydoc/expo/List/list.png)

3. **代码示例**

```js
import React from "react";
import {ListView, View, StyleSheet, Text} from "hippy-react"
import {mockData} from './UIStyles/mock';
import Style1 from './UIStyles/style1';
import Style2 from './UIStyles/style2';
import Style5 from './UIStyles/style5';
const STYLE_LOADING = 100;
const MAX_FETCH_TIMES = 50;

const styles = StyleSheet.create({
    container: {
        backgroundColor: "#ffffff"
    },
    itemContainer: {
        padding: 12,
    },
    spliter: {
        marginLeft: 12,
        marginRight: 12,
        height: 0.5,
        backgroundColor: "#e5e5e5"
    },
    loading: {
        fontSize: 11,
        color: "#aaaaaa",
        alignSelf: "center"
    }
});

export default class ListExample extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            dataSource: [],
            loadingState: '正在加载...'
        };
        this.fetchTimes = 0;
        this.mockFetchData = this.mockFetchData.bind(this);
    }

    mockFetchData() {
        if (this.fetchTimes++ >= MAX_FETCH_TIMES) {
            return Promise.resolve([]);
        }
        return new Promise(resolve => {
            setTimeout(() => {
                return resolve(mockData);
            }, 400);
        })
    }

    async componentWillMount() {
        let mockData = await this.mockFetchData();
        mockData.push({style: STYLE_LOADING});
        this.setState({dataSource: mockData});
    }

    onClickItem(index) {
        console.log('item :' + index + ' is clicked..');
    }

    getRenderRow(index) {
        let styleUI = null;
        let rowData = this.state.dataSource[index];
        let isLastItem = this.state.dataSource.length === index + 1;
        switch (rowData.style) {
            case 1:
                styleUI = <Style1 itemBean={rowData.itemBean} onClick={this.onClickItem.bind(this, index)}/>
                break;
            case 2:
                styleUI = <Style2 itemBean={rowData.itemBean} onClick={this.onClickItem.bind(this, index)}/>
                break;
            case 5:
                styleUI = <Style5 itemBean={rowData.itemBean} onClick={this.onClickItem.bind(this, index)}/>
                break;
            case STYLE_LOADING:
                styleUI = <Text style={styles.loading}>{this.state.loadingState}</Text>
        }
        return (
            <View style={styles.container}>
                <View style={styles.itemContainer}>
                    {styleUI}
                </View>
                {!isLastItem && <View style={styles.spliter}/>}
            </View>
        )
    }

    async onEndReached() {
        let {dataSource} = this.state;
        let newData = await this.mockFetchData();
        if (newData && newData.length) {
            let loading = dataSource[dataSource.length - 1];
            if (loading && loading.style === STYLE_LOADING) {
                dataSource.pop();
            }
            let newDataSource = dataSource.concat(newData);
            newDataSource.push(loading);
            this.setState({dataSource: newDataSource});
        } else {
            let loading = dataSource[dataSource.length - 1];
            if (loading && loading.style === STYLE_LOADING) {
                this.setState({loadingState: '没有更多数据'})
            }
        }
    }

    getRowType(index) {
        let self = this;
        let item = self.state.dataSource[index];
        return item.style;
    }

    render() {
        return (
            <ListView style={{flex: 1, backgroundColor: '#ffffff'}}
                      numberOfRows={this.state.dataSource.length}
                      renderRow={this.getRenderRow.bind(this)}
                      onEndReached={this.onEndReached.bind(this)}
                      getRowType={this.getRowType.bind(this)}
            />
        );
    }
}

```

