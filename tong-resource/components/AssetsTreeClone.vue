<template>
    <div class="assetsTree">
        <el-input
            class="assetsSearch"
            size='mini'
            placeholder="搜索..."
            v-model="filterText"
            >
        </el-input>
        <el-tree
            :data="resources.assets"
            node-key="id"
            ref="assetsTree"
            :highlight-current="true"
            :render-after-expand="false"
            @node-click="clickAssetFolder"
            :expand-on-click-node="false"
            :filter-node-method="filterNode"
            @node-drag-start="handleDragStart"
            @node-drag-enter="handleDragEnter"
            @node-drag-leave="handleDragLeave"
            @node-drag-over="handleDragOver"
            @node-drag-end="handleDragEnd"
            @node-drop="handleDrop"
            @node-contextmenu="rightContext"
            :render-content="renderContent"
            @node-expand="nodeExpand"
            @node-collapse="nodeCollapse"
            draggable
            :allow-drop="allowDrop"
            :allow-drag="allowDrag">
        </el-tree>
    </div>
</template>
<script>
import AssetsType from "../js/AssetsType"
import AssetsUtils from "../js/AssetsUtils"
let currId=0
export default {
    props:{
        resources:{
            type:Object,
            default(){
                return {}
            }
        }
    },
    data(){
        return {
            filterText: '',
            clickFolderInfo:{
                data:{},
                node:null,
                nodeCom:null,
                event:null
            },
            tree:null
        }
    },
    watch: {
        filterText(val) {
            this.tree.filter(val)
        }
    },
    mounted(){
        this.tree=this.$refs.assetsTree
        this.clickFromId(1)
    },
    methods: {
        //根据id进行节点点击,由于setCurrentKey没效果，重新封装
        clickFromId(id){
            function traverseNodeClick(children){
                for(let i=0;i<children.length;i++){
                    if(children[i].node.data.id===id){
                        return children[i].handleClick()
                    }
                    if(children[i].$children){
                        traverseNodeClick(children[i].$children)
                    }
                }
            }
            traverseNodeClick(this.tree.$children)
        },
        //根据节点对象进行节点点击,由于setCurrentNode没效果，重新封装
        clickFromNode(node){
            this.clickFromId(node.data.id)
        },
        //根据节点对象数据进行节点点击
        clickFromData(data){
            this.clickFromId(data.id)
        },
        //右键菜单
        rightContext(event,data,node,nodeCom){
            this.clickFolderInfo={event,data,node,nodeCom}
            this.$emit('rightContext',event,data,node,nodeCom)
        },
        dropDownItem(command){
            switch(command){
                case 'add':
                    AssetsUtils.AddFile.call(this,'未命名文件夹',this.clickFolderInfo)
                break
                case 'delete':
                    AssetsUtils.DeleteFile.call(this,this.clickFolderInfo)
                break
                case 'edit':
                    AssetsUtils.RenameFile.call(this,this.clickFolderInfo)
                break

            }
        },
        //点击文件夹
        clickAssetFolder(data,node,nodeCom){
            this.clickFolderInfo={event,data,node,nodeCom}
            this.$emit('click',data,node,nodeCom)
        },
        //搜索节点
        filterNode(value, data) {
            if(this.resources.onlyRenderFolder && data.type!=='folder') return false
            if (!value ) return true;
            return data.label.indexOf(value) !== -1
        },
        renderContent(h, { node, data }){
            //parent数据赋值与当前数据,可以更容易的知道父子级数据关系
            data.parent=node.parent.data
            //把assetsTree渲染的node赋予data
            data.assetNode=node
            //赋值nodecomponent
            //data.nodeCom=this.$refs.assetsTree
            // 默认配置类型为folder,通过set方式让type的改变可以随时被监视
            this.$set(data,'type',data.type || 'folder')
            !data.id && currId++
            data.id=data.id || currId
            //当前文件在服务端的状态:1未上传,2正在上传，3已上传，-1上传错误
            this.$set(data,'status',data.status || 3)
            //上传的进度，默认为0
            this.$set(data,'progress',data.progress || 0)
            // 类型首字母大写
            let typeKey = AssetsUtils.ToFirstUpperCase(data.type)
            if(this.resources.onlyRenderFolder && typeKey!=='Folder'){
                node.visible=false
                //console.log(node)
                return
            }
            //是否允许拖拽，默认为true
            data.allowDrag=data.allowDrag===undefined?true:data.allowDrag
            //默认的文件/文件夹图标
            let icon= AssetsType[typeKey].icon
            //默认的打开文件夹图标
            let openIcon=AssetsType['FolderOpen'].icon
            if(data.isOpen && typeKey==='Folder' && node.childNodes.length!==0){
                node.expanded=true
                icon=openIcon
                
            }
            //针对拖拽后，对默认文件夹的图标进行更改
            else if(typeKey==='Folder' && node.childNodes.length===0 && data.icon===openIcon){
                node.expanded=false
                data.isOpen=false
                data.icon=icon
                node.iconEle?node.iconEle.elm.className=icon:0
            }
            //设置默认icon
            data.icon=data.icon || icon
            let iconEle=(<i id='icon' class={data.icon} ></i>)
            let iconDiv=(
                <span>
                {iconEle}
                <span> </span>
                <span>{node.label}</span>
                </span>
                );
            node.iconEle=iconEle
            node.assetType=data.type
            node.allowDrag=data.allowDrag
            return iconDiv
        },
        nodeExpand(data,node){
            //首字母大写
            let typeKey = AssetsUtils.ToFirstUpperCase(data.type)
            if(data.type.toLowerCase()==='folder' 
            && data.icon===AssetsType[typeKey].icon){
                data.isOpen=true
                data.icon=AssetsType['FolderOpen'].icon
                node.iconEle.elm.className=data.icon
            }
        },
        nodeCollapse(data,node){
           
            let currType=AssetsUtils.ToFirstUpperCase(data.type)
            if(currType==='Folder' 
            && data.icon===AssetsType['FolderOpen'].icon){
                data.isOpen=false
                data.icon=AssetsType[currType].icon
                node.iconEle.elm.className=data.icon
            }
        }
        ,
        handleDragStart(node, ev) {
           
        },
        handleDragEnter(draggingNode, dropNode, ev) {
           
        },
        handleDragLeave(draggingNode, dropNode, ev) {
            
        },
        handleDragOver(draggingNode, dropNode, ev) {
            
        },
        handleDragEnd(draggingNode, dropNode, dropType, ev) {
           
        },
        handleDrop(draggingNode, dropNode, dropType, ev) {
            
        }
        ,
        allowDrop(draggingNode, dropNode) {
            let typeKey = AssetsUtils.ToFirstUpperCase(dropNode.assetType)
            if(typeKey!=='Folder'){
                return false
            }
            if(!this.isOutterDrag && dropNode.level===1){
                return false
            }
            return true
        },
        allowDrag(draggingNode) {
            return draggingNode.allowDrag
        }
    }
}
</script>
<style lang='stylus' scoped>
.assetsTree
    .assetsSearch
        input
            border-radius: 0px
    >>> .el-tree-node__content
        font-size 12.5px
    >>> .el-tree__empty-text
        font-size 13px
</style>

