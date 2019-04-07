<template>
    <el-card shadow="hover" body-style="padding: 0;">
        <el-tree :data="tree" :props="defaultProps" @node-click="handleNodeClick" :highlight-current="true"
                 :default-expand-all="true" :expand-on-click-node="false"></el-tree>
    </el-card>
</template>

<script>
    import Contents from 'contents';
    import $ from 'jquery';

    export default {
        name: "Toc",
        props: {
            readmeHtmlWithoutAnchor: {
                type: String,
                require: true,
            },
        },
        data() {
            return {
                tree: [],
                defaultProps: {
                    children: 'descendants',
                    label: 'name',
                },
                jumpToNodeId: '',
            }
        },

        methods: {
            // 点击树节点时跳转到锚点
            handleNodeClick(data) {
                window.location.href = '#' + data.id;
            },

            // 递归获取一维标题数组
            flattenTree(nodes) {
                let tree = [];

                nodes.forEach(node => {
                    tree.push({
                        id: node.id,
                        level: node.level,
                        name: _.trim(node.name),
                    });

                    if (node.descendants.length > 0) {
                        tree = _.concat(tree, this.flattenTree(node.descendants));
                    }
                });

                return tree;
            },
        },

        watch: {
            // 生成带 ID 属性的 readme.md 的 HTML
            readmeHtmlWithoutAnchor: function (n, o) {
                let html = $(`<div>${n}</div>`);

                let contents = Contents({
                    link: () => {
                    },
                    articles: html.find('h1,h2,h3,h4,h5,h6').get(),
                });

                this.tree = contents.tree();

                let articles = this.flattenTree(this.tree);
                articles.forEach(article => {
                    html.find(`h${article.level}`).each(function () {
                        if (_.trim($(this).text()) === article.name) {
                            $(this).attr('id', article.id);
                        }
                    });
                });

                this.$emit('render', html.html());
            },
        },
    }
</script>

<style scoped lang="scss">
    .el-card {
        margin-top: 45px;
        position: absolute;
        width: 300px;
        padding: 10px;
        overflow: hidden;
    }
</style>