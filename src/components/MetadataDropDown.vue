<template>
    <v-autocomplete
            :loading="isLoading"
            v-model="selected"
            :items="values"
            :item-text="rename"
            item-value="value"
            :label="labelTitle"
            multiple
            :disabled="searchDisabled"
    >
        <template slot="item" slot-scope="data">
            <span class="item-value-span"> {{ rename(data.item)}}</span>
            <span class="item-count-span">{{data.item.count}}</span>
        </template>
    </v-autocomplete>

    <!--<v-autocomplete-->
    <!--:value="getValue()"-->
    <!--@input="changedValue"-->
    <!--:items="values"-->
    <!--:item-text="rename"-->
    <!--:label="labelTitle"-->
    <!--multiple=""-->
    <!--&gt;</v-autocomplete>-->
</template>

<script>
    import {mapActions, mapState, mapGetters} from 'vuex'


    export default {
        name: "MetadataDropDown",
        props: {
            labelTitle: {type: String, required: true,},
            field: {type: String, required: true,},
        },
        data() {
            return {
                ageDialog: false,
                isLoading: true,
                values: [],//the list of values of the drop-down menu
            }
        },
        watch: {
            synonym() {
                this.loadData();
            },
            compound_query() {
                // if(!this.synonym)
                this.loadData();
            }
        },
        computed: {
            ...mapState([
                'query', 'synonym', "panelActive"
            ]),
            ...mapGetters({
                compound_query: 'build_query',
            }),
            selected: {
                get() {
                    // console.log("GET" + this.fullQuery[this.field]);
                    return this.query[this.field];
                },
                set(value) {
                    // console.log("SET" + value);
                    this.setDropDownSelected({field: this.field, list: value})
                }
            },
            searchDisabled() {
                return this.panelActive.length !== 0
            },
        },
        methods: {
            ...mapActions([
                'setDropDownSelected',
            ]),
            nthIndex(str, pat, n) {
                var L = str.length, i = -1;
                while (n-- && i++ < L) {
                    i = str.indexOf(pat, i);
                    if (i < 0) break;
                }
                return i;
            },
            //for the alternative solution
            // getValue() {
            //     console.log("GET" + this.fullQuery[this.field]);
            //
            //     return this.fullQuery[this.field];
            // },
            // changedValue(value) {
            //     console.log("SET" + value);
            //     this.setDropDownSelected({field: this.field, list: value})
            //     // this.$store.dispatch('setDropDownSelected', {field: this.field, list: ['TCGA']});
            // },
            rename(inp) {
                let value;
                if (inp.value !== null && inp.value !== undefined) {
                    if (this.field === 'dataset_name' && inp.value.length > 20) {
                        let i = this.nthIndex(inp.value, "_", 3);
                        value = inp.value.slice(0, i+1) + "\n" + inp.value.slice(i+1);
                    } else {
                        value = inp.value;
                    }
                } else
                    value = 'N/D(not defined)';

                let res;
                if (inp.count)
                    res = `${value}`;
                else
                    res = value;
                return res;
            }
            ,
            loadData() {
                const url = `field/${this.field}`;
                this.isLoading = true;

                // eslint-disable-next-line
                axios.post(url, this.compound_query)
                    .then((res) => {
                        return res.data
                    })
                    .then((res) => {
                        let vals = res.values
                        // console.log(res);
                        //to clean previously selected values
                        if (this.selected) {
                            // console.log(this.selected);
                            let zero_elements = this.selected.filter(value => !res.values.map(v => v.value).includes(value))
                                .sort().map(v => Object({
                                    value: v,
                                    count: 0
                                }));
                            // console.log(zero_elements);
                            vals = vals.concat(zero_elements);
                        }

                        this.values = vals;
                        this.isLoading = false;
                    });

                // console.log(this.selected.get().filter(v => this.values.indexOf(v) > -1));
//
                // this.selected.set(this.selected.get().filter(v => this.values.indexOf(v) > -1));
            }
            ,
        },
        created() {
            this.loadData();
        }
    }
</script>

<style scoped>
    .item-value-span {
        padding-right: 3.5em;
    }

    .item-count-span {
        /*float:right;*/
        position: absolute;
        right: 0.5em;
    }

</style>
