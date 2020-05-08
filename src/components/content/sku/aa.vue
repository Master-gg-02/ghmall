<template>

</template>

<script>
// 新建一个类Spu模拟从后端获取数据


class Spu {
    data = []

    constructor() {
        this.data = this.getData();
    }

    getData() {
        const data = sku_list; //这里的sku_list就是七月老师提供的数据中对应字段的数据，我们目前只需要这些就可以了
        return data
    }
}
export default {
    name: '',
    data() {
        return {
            sku: null,
            allSpecsList: [],
            selectable: null,
            selected: [],
            selectedItem: null,
            selectTips: '请选择：',
            // 1. 获取后端数据
            sku_list: [{
                "id": 2,
                "price": 77.76,
                "discount_price": null,
                "online": true,
                "img": "",
                "title": "金属灰·七龙珠",
                "spu_id": 2,
                "category_id": 17,
                "root_category_id": 3,
                "specs": [{
                        "key_id": 1,
                        "key": "颜色",
                        "value_id": 45,
                        "value": "金属灰"
                    },
                    {
                        "key_id": 3,
                        "key": "图案",
                        "value_id": 9,
                        "value": "七龙珠"
                    },
                    {
                        "key_id": 4,
                        "key": "尺码",
                        "value_id": 14,
                        "value": "小号 S"
                    }
                ],
                "code": "2$1-45#3-9#4-14",
                "stock": 5
            }],
            key_id: { // key_id 确定当前选了哪种类型的规格
                key: string, // 规格名
                key_id: number, // 规格id
                selectableList: { // 选择列表，用来描述我们上面画的表
                    value_id: { // 当前选择了哪个规格，上面key_id确定了规格种类，这里value_id确定了这种规格中的具体规格,
                        matchItems: { // 当前规格对应其他规格的匹配项
                            key_id: [ // 当前规格匹配的种类规格的种类id，
                                {
                                    key_id: numebr, // 种类id，这里为了方便数组中的直接取了后端返回的数据
                                    key: string, // 规格种类名称
                                    value_id: number, // 匹配项中的规格id
                                    value: string, //匹配项中的规格名称
                                },

                            ]

                        }

                    }

                }
            },
            key_id2: { //规格id
                key: string, // 规格名称 
                key_id: number, // 规格id
                value_list: { // 规格值列表 object
                    value_id: { // 规格值id
                        value_id: number, // 规格值id
                        value: string, // 规格值
                        selected: boolean, // 是否选中
                        disabled: boolean, // 是否不可选
                    }
                }
            }
        }
    },
    mounted(){
      currentSkuList = []
    },
    methods: {
      
        initData() {
            const sku = new Sku();
            const allSpecsList = sku.getAllSpecsList();
            const selectable = sku.getSelectable();
            this.setData({
                allSpecsList,
                selectable,
                sku
            });
        },

        _getCurrentSkuList() {
            this.currentSkuList = this.data.map(item => item.specs);
        },
        _transMatrix() {
            this._getCurrentSkuList();

            let transResult = {};

            this.currentSkuList.forEach(specs => {
                specs.forEach(item => {
                    if (!transResult[item['key_id']]) {
                        transResult[item['key_id']] = {
                            key_id: item['key_id'],
                            key: item['key'],
                            value_list: {
                                [item['value_id']]: {
                                    value_id: item['value_id'],
                                    value: item['value'],
                                    selected: false,
                                    disabled: false
                                }
                            }
                        }
                    } else if (!transResult[item['key_id']].value_list[item['value_id']]) {
                        transResult[item['key_id']].value_list[item['value_id']] = {
                            value_id: item['value_id'],
                            value: item['value'],
                            selected: false,
                            disabled: false
                        }
                    }
                })
            })
            return transResult;
        },

        getAllSpecsList() {
            const transResult = this._transMatrix();
            this.allSpecsList = Object.keys(transResult).map(key => {
                let obj = JSON.parse(JSON.stringify(transResult[key]));
                obj.value_list = Object.keys(obj.value_list).map(vk => obj.value_list[vk]);
                return obj;
            })
            return this.allSpecsList;
        },

        getSelectable() {
            if (this.allSpecsList.length === 0) {
                this.allSpecsList()
            }

            if (this.currentSkuList.length === 0) {
                this.currentSkuList = this.data.map(item => item.specs);
            }

            const rowLength = this.allSpecsList.length;

            for (let row = 0; row < rowLength; row++) {
                let {
                    key_id,
                    key
                } = this.allSpecsList[row];
                let columnList = this.allSpecsList[row].value_list;
                this.selectable[key_id] = {
                    key_id,
                    key,
                    selectableList: {}
                }
                for (let column = 0; column < columnList.length; column++) {
                    let {
                        value_id,
                        value
                    } = columnList[column];
                    this.selectable[key_id].selectableList[value_id] = {
                        value_id,
                        value,
                        matchItems: null
                    }
                }

                this.currentSkuList.forEach(specificSpecs => {
                    let matchItems = {};
                    let currentVlaueId = '';
                    specificSpecs.forEach(specsItem => {
                        if (specsItem.key_id !== key_id) {
                            matchItems[specsItem.key_id] = [specsItem]
                        } else {
                            currentVlaueId = specsItem.value_id;
                        }
                    })

                    if (!this.selectable[key_id].selectableList[currentVlaueId].matchItems) {
                        this.selectable[key_id].selectableList[currentVlaueId].matchItems = matchItems;
                    } else {
                        Object.keys(this.selectable[key_id].selectableList[currentVlaueId].matchItems).forEach(k => {
                            this.selectable[key_id].selectableList[currentVlaueId].matchItems[k].push(...matchItems[k])
                        })
                    }

                })
            }

            return this.selectable;
        }

    }
}
</script>

<style>

</style>
