我去收拾旧物，一堆玩具士兵在排兵列阵。
我问:“你们在做什么？”
士兵回答:“我们在等司令回来。”
我一愣，犹豫了一下，说:“你们的司令不会回来了。”
“司令死了吗？”士兵们问。
“不，他只是长大了。


ipone开启飞行模式一天只会消耗3%的电。
乌龟的心脏离开身体后，还能跳动四个小时，所有的北极熊都是左撇子。
人一直看着自己的手心的话，手心会害羞发热 你看，我知道好多奇怪的事情啊，却永远不知道怎么让你喜欢我。

/*
 * 添加/修改金币栏位
 */
Ext.define('MyApp.view.GoodsTypeGameWindow', {
    extend: 'Ext.window.Window',
    width: 720,
    height: 400,
    bodyStyle:'overflow-y:auto;overflow-x:hidden;',
    title: '新增/修改商品类型游戏关联',
    closeAction: 'hide',
    modal: true,
    form: null,
    getForm: function(){
        var me = this;
        if(me.form==null){
            me.form = Ext.widget('form',{
                layout: 'column',
                defaults: {
                    margin: '5 5 5 5',
                    labelWidth: 100,
                    xtype: 'textfield'
                },
                items: [{
                    xtype: 'gameselectorgame',
                    itemId : 'MyApp_view_goods_gamelink_ID',
                    columnWidth: 1,
                    fieldLabel: '游戏属性',
                    labelWidth: 80,
                    allowBlank: false
                },DataDictionary.getDataDictionaryCombo('regionExist', {
                    fieldLabel: '是否在iOS上显示',
                    name: 'isShowOnIos',
                    columnWidth: 0.4,
                    labelWidth: 130,
                    allowBlank: false
                }),DataDictionary.getDataDictionaryCombo('regionExist', {
                    fieldLabel: '是否在Android上显示',
                    name: 'isShowOnAndroid',
                    columnWidth: 0.4,
                    labelWidth: 150,
                    allowBlank: false
                }),DataDictionary.getDataDictionaryCombo('goodsTradeType',{
                    fieldLabel: '交易类型',
                    name: 'tradeType',
                    editable: false,
                    labelWidth: 80,
                    columnWidth: .5
                }),DataDictionary.getDataDictionaryCheckboxGroup('mainGoodsType',{
                    fieldLabel: '商品类型',
                    name: 'goodsType',
                    itemCls: 'x-check-group-alt',
                    columnWidth: 1,
                    maxHeight: 40,
                    allowBlank: false,
                    columns: 4
                })],
                buttons: [{
                    text:'保存',
                    formBind: true,
                    disabled: true,
                    handler: function() {
                        var tab = Ext.getCmp('userManager'),
                            form = me.getForm(),
                            record = form.getRecord(),
                            url = './rs/goodsTypeGame/addGoodsTypeGame',
                            params,
                            message = '新增',
                            gameId = form.getForm().findField('gameName'),
                            tradeType = form.getForm().findField('tradeType');
                        if(!form.isValid()){
                            return;
                        }

                        var values = form.getValues();
                        gameId = gameId.getValue();
                        if(gameId==null || gameId==""){
                            Ext.ux.Toast.msg("温馨提示", "请选择游戏属性");
                            return;
                        }
                        var myCheckboxGroup = form.getForm().findField('goodsType');
                        var cbitems = myCheckboxGroup.items;
                        var ids = [];
                        for (var i = 0; i < cbitems.length; i++) {
                            if (cbitems.get(i).checked) {
                                ids.push(cbitems.get(i).name);
                            }
                        }
                        if(ids.length==0){
                            Ext.ux.Toast.msg("温馨提示", "请选择商品类型");
                            return;
                        }

                        // console.log(tradeType.getValue());
                        if(tradeType.getValue() == null) {
                            tradeType = 0;
                            form.getForm().findField('tradeType').setValue(0);
                        } else {
                            tradeType = tradeType.getValue();
                        }

                        form.updateRecord(record);
                        // console.log(params);
                        params = {
                            'gameId':gameId,
                            'isShowOnIos':values.isShowOnIos,
                            'isShowOnAndroid':values.isShowOnAndroid,
                            'goodsType':ids,
                            'tradeType':tradeType
                        };
                        if(me.isUpdate){
                            url = './rs/goodsTypeGame/modifyGoodsTypeGame';
                            message = '修改';
                            Ext.Object.merge(params,{
                                'id': record.get('id')
                            });
                        }
                        // console.log(params);
                        form.submit({
                            url : url,
                            params: params,
                            success : function(from, action, json) {
                                var goldRowManager = Ext.getCmp('goodsTypeGameManager'),
                                    store = goldRowManager.getStore();
                                Ext.ux.Toast.msg("温馨提示", message + "成功");
                                me.close();
                                store.load();
                            },
                            exception : function(from, action, json) {
                                Ext.ux.Toast.msg("温馨提示", json.responseStatus.message);
                            }
                        });
                    }
                }]
            });
        }
        return this.form;
    },
    isUpdate: null,
    bindData: function(record,isUpdate){
        var me = this,
            form = me.getForm().getForm(),
            gameProp = me.getForm().getComponent('MyApp_view_goods_gamelink_ID');
        form.reset();
        form.loadRecord(record);
        me.isUpdate = isUpdate;
        if(!isUpdate){
            gameProp.setDisabled(false);
            form.reset();
        }
        else{
            gameProp.setDisabled(true);
        }
    },
    initComponent: function() {
        var me = this;
        Ext.applyIf(me, {
            items: [me.getForm()]
        });
        me.callParent(arguments);
    }
});

/**
 * 编辑交易说明
 * wfr 2017.09.07 新增
 */
Ext.define('MyApp.view.EditWindow', {
    extend: 'Ext.window.Window',
    width: 800,
    height: 500,
    title: '编辑',
    closeAction: 'hide',
    modal: true,
    form: null,
    getForm: function(){
        var me = this;
        if(me.form==null){
            me.form = Ext.widget('form',{
                layout: 'column',
                defaults: {
                    margin: '10 10 10 10',
                    labelWidth: 100,
                    xtype: 'textfield'
                },
                items: [
                    {
                        xtype: 'hidden',
                        name: 'id'
                    },{
                    name: 'gameName',
                    fieldLabel: '游戏名称',
                    columnWidth: .85,
                    allowBlank: false
                },DataDictionary.getDataDictionaryCombo('mainGoodsType',{
                    fieldLabel: '商品类型',
                    name: 'goodsType',
                    readOnly: true,
                    editable: false,
                    labelWidth: 100,
                    columnWidth: .85
                }),{
                    name: 'tradeDescription',
                    fieldLabel: '交易说明',
                    xtype: 'htmleditor',
                    columnWidth: .85,
                    height: 312,
                    allowBlank: false
                }],
                buttons: [{
                    text:'保存',
                    formBind: true,
                    disabled: true,
                    handler: function() {
                        var formView = me.getForm(),
                            form = formView.getForm();
                        console.log(form);
                        var id = form.findField('id').getValue(),
                            url = './rs/goodsTypeGame/editDescription',
                            params = {
                                'id':id,
                            };
                        form.submit({
                            url : url,
                            params: params,
                            success : function(from, action, json) {
                                var goldRowManager = Ext.getCmp('goodsTypeGameManager'),
                                    store = goldRowManager.getStore();
                                Ext.ux.Toast.msg("温馨提示", "编辑成功");
                                me.close();
                                store.load();
                            },
                            exception : function(from, action, json) {
                                Ext.ux.Toast.msg("温馨提示", json.responseStatus.message);
                                me.close();
                            }
                        });
                    }
                }]
            });
        }
        return this.form;
    },
    bindData: function (record) {
        var me = this,
            form = me.getForm().getForm();
        form.loadRecord(record);
    },
    initComponent: function() {
        var me = this;
        Ext.applyIf(me, {
            items: [me.getForm()]
        });
        me.callParent(arguments);
    }
});

/*
 * 配置商品类型关联游戏交易类型信息
 * sunjianlin 2017.02.20 新增
 */
Ext.define('MyApp.view.setTradeTypeWindow', {
    extend: 'Ext.window.Window',
    title: '配置交易类型',
    width: 400,
    closeAction: 'hide',
    modal: true,
    store: null,
    form: null,
    getForm: function () {
        var me = this;
        if (me.form == null) {
            me.form = Ext.widget('form', {
                layout: 'column',
                defaults: {
                    margin: '10 10 5 5',
                },
                items: [{
                    xtype: 'textfield',
                    readOnly: true,
                    allowBlank: false,
                    columnWidth: .8,
                    labelWidth: 100,
                    fieldLabel: '游戏',
                    name: 'gameName'
                }, DataDictionary.getDataDictionaryCombo('mainGoodsType',{
                    fieldLabel: '商品类型',
                    name: 'goodsType',
                    readOnly: true,
                    editable: false,
                    labelWidth: 100,
                    columnWidth: .8
                }),DataDictionary.getDataDictionaryCombo('goodsTradeType',{
                    fieldLabel: '交易类型',
                    name: 'tradeType',
                    editable: false,
                    labelWidth: 100,
                    columnWidth: .8
                })],
                buttons: [{
                    text: '保存',
                    handler: function () {
                        var form = me.getForm().getForm();
                        if (!form.isValid()) {
                            return;
                        }
                        var record = form.getRecord(),
                            store = Ext.getCmp('goodsTypeGameManager').getStore();
                        var goodsType = form.findField('goodsType');
                        console.log(goodsType);
                        var tradeType = form.findField('tradeType');
                        var params = {
                            'goodsType':goodsType,
                            'tradeType':tradeType,
                            'id':record.get('id')
                        }
                        form.submit({
                            url: './rs/goodsTypeGame/modifyGoodsTypeGame',
                            method: 'POST',
                            params: params,
                            success: function (from, action, json) {
                                Ext.ux.Toast.msg("温馨提示", "配置成功");
                                store.load();
                                me.close();
                            },
                            exception: function (from, action, json) {
                                Ext.ux.Toast.msg("温馨提示", json.responseStatus.message);
                            }
                        });
                        me.close();

                    }
                }]
            });
        }
        return me.form;
    },
    bindData: function (record) {
        var me = this,
            form = me.getForm().getForm();
        form.loadRecord(record);
        var goodsType = form.findField('goodsType');
        console.log(goodsType);
        // form.findField('goodsType').setValue(DataDictionary.rendererSubmitToDisplay(goodsType,'mainGoodsType'));
    },
    constructor: function (config) {
        var me = this, cfg = Ext.apply({}, config),
            form = me.getForm();
        me.items = [me.getForm()];
        me.callParent([cfg]);
    },
});




/*
 * 商品类型关联游戏信息管理页面
 */
Ext.define('MyApp.view.goodsTypeGameManager', {
    extend: 'Ext.panel.Panel',
    id: 'goodsTypeGameManager',
    closable: true,
    title: '商品类型关联游戏信息',
    toolbar: null,
    getToolbar: function(){
        var me = this;
        if(Ext.isEmpty(me.toolbar)){
            me.toolbar = Ext.widget('toolbar',{
                dock: 'top',
                items: [{
                    text: '新增',
                    handler: function(){
                        me.addGoodsTypeGame();
                    }
                },'-',{
                    text: '编辑',
                    handler: function(){
                        me.editDescription();
                    }
                },'-',{
                    text: '删除',
                    handler: function(){
                        me.deleteGoodsTypeGame();
                    }
                },'-',{
                    text: '在Android上显示',
                    handler: function(){
                        me.showOnAndroid();
                    }
                },'-',{
                    text: '禁止在Android上显示',
                    handler: function(){
                        me.unShowOnAndroid();
                    }
                },'-',{
                    text: '在IOS上显示',
                    handler: function(){
                        me.showOnIos();
                    }
                },'-',{
                    text: '禁止在IOS上显示',
                    handler: function(){
                        me.unShowOnIos();
                    }
                },
                //     '-',{
                //     text: '设置交易类型',
                //     handler: function(){
                //         me.setTradeType();
                //     }
                // },
                    '-',{
                    text: '开通出售',
                    handler: function(){
                        me.openSaleInM();
                    }
                },'-',{
                    text: '关闭出售',
                    handler: function(){
                        me.closeSaleInM();
                    }
                },'-',{
                    text: 'M发布单上架同步',
                    handler: function(){
                        me.upSynchronized();
                    }
                },'-',{
                        text: 'M发布单下架同步',
                        handler: function(){
                            me.downSynchronized();
                        }
                },'-',{
                        text: '批量更新阿里云图片',
                        handler: function(){
                            me.upSynchronizedImgList();
                        }
                },'-',{
                    text: '配置交易类型',
                    handler: function(){
                        me.setTradeType();
                    }
                }]
            });
        }
        return me.toolbar;
    },
    goodsWindow: null,
    getGoodsWindow: function(){
        if(this.goodsWindow == null){
            this.goodsWindow = new MyApp.view.goodsWindow();
        }
        return this.goodsWindow;
    },
    setTradeTypeWindow: null,
    getSetTradeTypeWindow: function(){
        if(this.setTradeTypeWindow == null){
            this.setTradeTypeWindow = new MyApp.view.setTradeTypeWindow();
        }
        return this.setTradeTypeWindow;
    },
    goodsTypeGameWindow: null,
    getGoodsTypeGameWindow: function(){
        if(this.goodsTypeGameWindow == null){
            this.goodsTypeGameWindow = new MyApp.view.GoodsTypeGameWindow();
        }
        return this.goodsTypeGameWindow;
    },
    //编辑窗口
    editWindow: null,
    getEditWindow: function(){
        if(this.editWindow == null){
            this.editWindow = new MyApp.view.EditWindow();
        }
        return this.editWindow;
    },

    addGoodsTypeGame: function() {
        var window = this.getGoodsTypeGameWindow();
        window.bindData(Ext.create('MyApp.model.GoodsTypeGameModel'),false);
        window.show();
    },

    //编辑交易说明
    editDescription: function() {
        var me = this,
            selModel = me.getGoodsTypeGameGrid().getSelectionModel(),
            selRecords = selModel.getSelection(),
            window = me.getEditWindow();
        if(selRecords == null||selRecords.length<=0){
            Ext.ux.Toast.msg("温馨提示", "请先选择要编辑的信息");
            return;
        }
        var theForm = window.getForm().getForm();
        theForm.findField("gameName").setReadOnly(true);
        theForm.findField("goodsType").setReadOnly(true);
        window.bindData(selRecords[0], true);
        window.show();

    },
    
    deleteGoodsTypeGame:function(){
        var grid = this.getGoodsTypeGameGrid(),
            selModel = grid.getSelectionModel(),
            selRecords = selModel.getSelection();
        if(selRecords == null||selRecords.length<=0){
            Ext.ux.Toast.msg("温馨提示", "请先选择要删除的信息");
            return;
        }

        Ext.MessageBox.confirm('温馨提示', '确定删除该信息吗？', function(btn){
            if(btn == 'yes'){
                Ext.Ajax.request({
                    url : './rs/goodsTypeGame/deleteGoodsTypeGame',
                    params: {'id':selRecords[0].get("id"),'gameId':selRecords[0].get("gameId"),'goodsType':selRecords[0].get("goodsType")},
                    success : function(response, opts) {
                        Ext.ux.Toast.msg("温馨提示", "删除成功！");
                        grid.getStore().load();
                        grid.getSelectionModel().deselectAll();
                    },
                    exception : function(response, opts) {
                        var json = Ext.decode(response.responseText);
                        Ext.ux.Toast.msg("温馨提示", json.responseStatus.message);
                    }
                });
            }else{
                return;
            }
        });
    },
    /**
     * 在iOS上显示
     */
    showOnIos : function() {
        var grid = this.getGoodsTypeGameGrid(),
            selModel = grid.getSelectionModel(),
            selRecords = selModel.getSelection();
        if(selRecords == null||selRecords.length<=0){
            Ext.ux.Toast.msg("温馨提示", "请先选择要在iOS上显示的商品类型");
            return;
        }
        if(selRecords[0].get("isShowIos")){
            Ext.ux.Toast.msg("温馨提示", "当前商品类型已可以在iOS上显示");
            return;
        }
        Ext.MessageBox.confirm('温馨提示', '确认开通？', function(btn){
            if(btn == 'yes'){
                Ext.Ajax.request({
                    url : './rs/goodsTypeGame/showOnIOS',
                    params: {
                        'id':selRecords[0].get("id"),
                        'gameId':selRecords[0].get("gameId"),
                        'goodsType':selRecords[0].get("goodsType"),
                        'salable':selRecords[0].get("salable")},
                    success : function(response, opts) {
                        Ext.ux.Toast.msg("温馨提示", "操作成功！");
                        grid.getStore().load();
                        grid.getSelectionModel().deselectAll();
                    },
                    exception : function(response, opts) {
                        var json = Ext.decode(response.responseText);
                        Ext.ux.Toast.msg("温馨提示", json.responseStatus.message);
                    }
                });
            }else{
                return;
            }
        });
    },
    /**
     * 在Android上显示
     */
    showOnAndroid : function() {
        var grid = this.getGoodsTypeGameGrid(),
            selModel = grid.getSelectionModel(),
            selRecords = selModel.getSelection();
        if(selRecords == null||selRecords.length<=0){
            Ext.ux.Toast.msg("温馨提示", "请先选择要在Android上显示的商品类型");
            return;
        }
        if(selRecords[0].get("isShowAndroid")){
            Ext.ux.Toast.msg("温馨提示", "当前商品类型已可以在Android上显示");
            return;
        }
        Ext.MessageBox.confirm('温馨提示', '确认开通？', function(btn){
            if(btn == 'yes'){
                Ext.Ajax.request({
                    url : './rs/goodsTypeGame/showOnAndroid',
                    params: {
                        'id':selRecords[0].get("id"),
                        'gameId':selRecords[0].get("gameId"),
                        'goodsType':selRecords[0].get("goodsType"),
                        'salable':selRecords[0].get("salable")},
                    success : function(response, opts) {
                        Ext.ux.Toast.msg("温馨提示", "操作成功！");
                        grid.getStore().load();
                        grid.getSelectionModel().deselectAll();
                    },
                    exception : function(response, opts) {
                        var json = Ext.decode(response.responseText);
                        Ext.ux.Toast.msg("温馨提示", json.responseStatus.message);
                    }
                });
            }else{
                return;
            }
        });
    },
    openSaleInM : function() {
        var grid = this.getGoodsTypeGameGrid(),
            selModel = grid.getSelectionModel(),
            selRecords = selModel.getSelection();
        if(selRecords == null||selRecords.length<=0){
            Ext.ux.Toast.msg("温馨提示", "请先选择要开通的商品类型");
            return;
        }
        if(selRecords[0].get("salable")){
            Ext.ux.Toast.msg("温馨提示", "当前商品类型已开通出售");
            return;
        }
        Ext.MessageBox.confirm('温馨提示', '确认开通？', function(btn){
            if(btn == 'yes'){
                Ext.Ajax.request({
                    url : './rs/goodsTypeGame/openSaleInM',
                    params: {
                        'id':selRecords[0].get("id"),
                        'gameId':selRecords[0].get("gameId"),
                        'goodsType':selRecords[0].get("goodsType")},
                    success : function(response, opts) {
                        Ext.ux.Toast.msg("温馨提示", "开通成功！");
                        grid.getStore().load();
                        grid.getSelectionModel().deselectAll();
                    },
                    exception : function(response, opts) {
                        var json = Ext.decode(response.responseText);
                        Ext.ux.Toast.msg("温馨提示", json.responseStatus.message);
                    }
                });
            }else{
                return;
            }
        });
    },
    updateGoodsSource: function (button, e, eOpts) {
        var me = this,
            forme = me.getQueryForm().getValues(),
            createStartTime =forme.createStartTime,
            createEndTime = forme.createEndTime;
        var startTime = new Date(createStartTime).getTime(),
            endTime = new Date(createEndTime).getTime();
        if((endTime - startTime)>15*24*60*60*1000 ){
            Ext.ux.Toast.msg("温馨提示", "同步时间间隔过长");
            return;
        }
        var myMask = new Ext.LoadMask(Ext.getBody(), {//Ext.getBody()也可以是Ext.getCmp('').getEl()窗口名称
            msg    : "正在进行商品来源同步,请稍后...",//你要写成Loading...也可以
            msgCls : 'z-index:10000;'
        });
        myMask.show();
        var parames ={
            "createStartTime":createStartTime,
            "createEndTime":createEndTime
        }
        Ext.Ajax.request({
            url: './rs/goodsTypeGame/updateGoodsSource',
            params: parames,
            method: 'POST',
            success: function (response, opts) {
                Ext.ux.Toast.msg("商品来源同步成功");
                myMask.hide();
            },
            exception: function (response, opts) {
                var json = Ext.decode(response.responseText);
                Ext.ux.Toast.msg("温馨提示", json.responseStatus.message);
                myMask.hide();
            }
        });
    },
    /**
     * 下架同步
     * @param button
     * @param e
     * @param eOpts
     */
    downSynchronized: function (button, e, eOpts) {
        var me = this,
            forme = me.getQueryForm().getValues(),
            createStartTime =forme.createStartTime,
            createEndTime = forme.createEndTime;
        var startTime = new Date(createStartTime).getTime(),
            endTime = new Date(createEndTime).getTime();
        if((endTime - startTime)>30*24*60*60*1000 ){
            Ext.ux.Toast.msg("温馨提示", "同步时间间隔过长(每次选择一个月)");
            return;
        }
        var myMask = new Ext.LoadMask(Ext.getBody(), {//Ext.getBody()也可以是Ext.getCmp('').getEl()窗口名称
            msg    : "同步路漫漫，再给我两分钟...",//你要写成Loading...也可以
            msgCls : 'z-index:10000;'
        });
        myMask.show();
        var parames ={
            "createStartTime":createStartTime,
            "createEndTime":createEndTime
        }
        Ext.Ajax.request({
            url: './rs/goodsTypeGame/downSynchronized',
            params: parames,
            method: 'POST',
            success: function (response, opts) {
                Ext.ux.Toast.msg("下架同步成功");
                myMask.hide();
            },
            exception: function (response, opts) {
                var json = Ext.decode(response.responseText);
                Ext.ux.Toast.msg("温馨提示", json.responseStatus.message);
                myMask.hide();
            }
        });
    },
    /**
     * 上架同步
     * @param button
     * @param e
     * @param eOpts
     */
    upSynchronized: function (button, e, eOpts) {
        var me = this,
            forme = me.getQueryForm().getValues(),
            createStartTime =forme.createStartTime,
            createEndTime = forme.createEndTime;
        var startTime = new Date(createStartTime).getTime(),
            endTime = new Date(createEndTime).getTime();
        if((endTime - startTime)>30*24*60*60*1000 ){
            Ext.ux.Toast.msg("温馨提示", "同步时间间隔过长(每次选择一个月)");
            return;
        }
        var myMask = new Ext.LoadMask(Ext.getBody(), {//Ext.getBody()也可以是Ext.getCmp('').getEl()窗口名称
            msg    : "快上马,架！架！架！...",//你要写成Loading...也可以
            msgCls : 'z-index:10000;'
        });
        myMask.show();
        var parames ={
            "createStartTime":createStartTime,
            "createEndTime":createEndTime
        }
        Ext.Ajax.request({
            url: './rs/goodsTypeGame/upSynchronized',
            params: parames,
            method: 'POST',
            success: function (response, opts) {
                Ext.ux.Toast.msg("上架同步成功");
                myMask.hide();
            },
            exception: function (response, opts) {
                var json = Ext.decode(response.responseText);
                Ext.ux.Toast.msg("温馨提示", json.responseStatus.message);
                myMask.hide();
            }
        });
    },
    /**
     * 批量更新阿里云商品图片列表字段 author fangcongpeng
     * 2017-06-27
     * @param button
     * @param e
     * @param eOpts
     */
    upSynchronizedImgList: function (button, e, eOpts) {
        var me = this,
            forme = me.getQueryForm().getValues(),
            createStartTime =forme.createStartTime,
            createEndTime = forme.createEndTime;
        var startTime = new Date(createStartTime).getTime(),
            endTime = new Date(createEndTime).getTime();
        if((endTime - startTime)>15*24*60*60*1000 ){
            Ext.ux.Toast.msg("温馨提示", "更新时间间隔过长");
            return;
        }
        var myMask = new Ext.LoadMask(Ext.getBody(), {//Ext.getBody()也可以是Ext.getCmp('').getEl()窗口名称
            msg    : "正在进行更新,请稍后...",//你要写成Loading...也可以
            msgCls : 'z-index:10000;'
        });
        myMask.show();
        /*var parames ={
            "createStartTime":createStartTime,
            "createEndTime":createEndTime
        }*/
        Ext.Ajax.request({
            url: './rs/goodsTypeGame/upSynchronizedImgList',
            /*params: parames,*/
            method: 'POST',
            success: function (response, opts) {
                Ext.ux.Toast.msg("批量更新成功");
                myMask.hide();
            },
            exception: function (response, opts) {
                var json = Ext.decode(response.responseText);
                Ext.ux.Toast.msg("温馨提示", json.responseStatus.message);
                myMask.hide();
            }
        });
    },
    /**
     * 禁止在iOS上显示
     */
    unShowOnIos : function() {
        var grid = this.getGoodsTypeGameGrid(),
            selModel = grid.getSelectionModel(),
            selRecords = selModel.getSelection();
        if(selRecords == null||selRecords.length<=0){
            Ext.ux.Toast.msg("温馨提示", "请先选择要禁止在iOS上显示的商品类型");
            return;
        }
        if(!selRecords[0].get("isShowIos")){
            Ext.ux.Toast.msg("温馨提示", "当前商品类型以禁止在iOS上显示");
            return;
        }
        Ext.MessageBox.confirm('温馨提示', '确认关闭？', function(btn){
            if(btn == 'yes'){
                Ext.Ajax.request({
                    url : './rs/goodsTypeGame/unShowOnIOS',
                    params: {
                        'id':selRecords[0].get("id"),
                        'gameId':selRecords[0].get("gameId"),
                        'goodsType':selRecords[0].get("goodsType"),
                        'salable':selRecords[0].get("salable")},
                    success : function(response, opts) {
                        Ext.ux.Toast.msg("温馨提示", "关闭成功！");
                        grid.getStore().load();
                        grid.getSelectionModel().deselectAll();
                    },
                    exception : function(response, opts) {
                        var json = Ext.decode(response.responseText);
                        Ext.ux.Toast.msg("温馨提示", json.responseStatus.message);
                    }
                });
            }else{
                return;
            }
        });
    },
    /**
     * 禁止在Android上显示
     */
    unShowOnAndroid : function() {
        var grid = this.getGoodsTypeGameGrid(),
            selModel = grid.getSelectionModel(),
            selRecords = selModel.getSelection();
        if(selRecords == null||selRecords.length<=0){
            Ext.ux.Toast.msg("温馨提示", "请先选择要禁止在Android上显示的商品类型");
            return;
        }
        if(!selRecords[0].get("isShowAndroid")){
            Ext.ux.Toast.msg("温馨提示", "当前商品类型以禁止在Android上显示");
            return;
        }
        Ext.MessageBox.confirm('温馨提示', '确认禁止在Android上显示？', function(btn){
            if(btn == 'yes'){
                Ext.Ajax.request({
                    url : './rs/goodsTypeGame/unShowOnAndroid',
                    params: {
                        'id':selRecords[0].get("id"),
                        'gameId':selRecords[0].get("gameId"),
                        'goodsType':selRecords[0].get("goodsType"),
                        'salable':selRecords[0].get("salable")},
                    success : function(response, opts) {
                        Ext.ux.Toast.msg("温馨提示", "禁止在Android上显示操作成功！");
                        grid.getStore().load();
                        grid.getSelectionModel().deselectAll();
                    },
                    exception : function(response, opts) {
                        var json = Ext.decode(response.responseText);
                        Ext.ux.Toast.msg("温馨提示", json.responseStatus.message);
                    }
                });
            }else{
                return;
            }
        });
    },
    closeSaleInM : function() {
        var grid = this.getGoodsTypeGameGrid(),
            selModel = grid.getSelectionModel(),
            selRecords = selModel.getSelection();
        if(selRecords == null||selRecords.length<=0){
            Ext.ux.Toast.msg("温馨提示", "请先选择要关闭的商品类型");
            return;
        }
        if(!selRecords[0].get("salable")){
            Ext.ux.Toast.msg("温馨提示", "当前商品类型已关闭出售");
            return;
        }
        Ext.MessageBox.confirm('温馨提示', '确认关闭？', function(btn){
            if(btn == 'yes'){
                Ext.Ajax.request({
                    url : './rs/goodsTypeGame/closeSaleInM',
                    params: {
                        'id':selRecords[0].get("id"),
                        'gameId':selRecords[0].get("gameId"),
                        'goodsType':selRecords[0].get("goodsType")},
                    success : function(response, opts) {
                        Ext.ux.Toast.msg("温馨提示", "关闭成功！");
                        grid.getStore().load();
                        grid.getSelectionModel().deselectAll();
                    },
                    exception : function(response, opts) {
                        var json = Ext.decode(response.responseText);
                        Ext.ux.Toast.msg("温馨提示", json.responseStatus.message);
                    }
                });
            }else{
                return;
            }
        });
    },

    setTradeType: function () {
        var me = this,
            selModel = me.getGoodsTypeGameGrid().getSelectionModel(),
            selRecords = selModel.getSelection(),
            window = me.getSetTradeTypeWindow();
        if (selRecords == null || selRecords.length <= 0) {
            Ext.ux.Toast.msg("温馨提示", "请先选择要配置的游戏商品类型");
            return;
        }
        window.bindData(selRecords[0]);
        window.show();
    },

    queryForm: null,
    getQueryForm: function(){
        var me = this;
        if(me.queryForm==null){
            me.queryForm = Ext.widget('form',{
                layout: 'column',
                defaults: {
                    margin: '10 10 10 10',
                    columnWidth: .2,
                    labelWidth: 80,
                    xtype: 'textfield'
                },
                items: [{
                    xtype: 'gameselectorgame',
                    itemId : 'MyApp_view_goods_gamelink_ID',
                    columnWidth: .5,
                    allowBlank: true,
                    fieldLabel: '游戏属性'
                } , DataDictionary.getDataDictionaryCombo('mainGoodsType', {
                        fieldLabel: '商品类型',
                        labelWidth: 80,
                        columnWidth: .5,
                        name: 'goodsType',
                        editable: false,
                        value: 0
                    },{value: 0, display: '全部'}),
                    {
                        fieldLabel: '同步时间',
                        xtype: 'rangedatefield',
                        fromName: 'createStartTime',
                        toName: 'createEndTime',
                        fromValue: new Date(),
                        toValue: new Date(),
                        columnWidth: .35,
                    },
                ],
                buttons: [{
                    text:'查询',
                    handler: function() {
                        me.getPagingToolbar().moveFirst();
                    }
                },'-',{
                    text:'重置',
                    handler: function() {
                        me.getQueryForm().getForm().reset();
                    }
                },'->']
            });
        }
        return this.queryForm;
    },
    pagingToolbar: null,
    getPagingToolbar: function(){
        var me = this;
        if(me.pagingToolbar==null){
            me.pagingToolbar = Ext.widget('pagingtoolbar',{
                store: me.getStore(),
                dock: 'bottom',
                displayInfo: true
            });
        }
        return me.pagingToolbar;
    },
    store: null,
    getStore: function(){
        var me = this;
        if(me.store==null){
            me.store = Ext.create('MyApp.store.GoodsTypeGameStore',{
                autoLoad: true,
                listeners: {
                    beforeload : function(store, operation, eOpts) {
                        var queryForm = me.getQueryForm(),
                            gameId = queryForm.getForm().findField('gameName'),
                            goodsType = queryForm.getForm().findField('goodsType'),
                            tradeType = queryForm.getForm().findField('tradeType');
                        if (queryForm != null) {
                            var values = queryForm.getValues();
                            gameId = gameId.getValue();
                            goodsType = goodsType.getValue();
                            Ext.apply(operation, {
                                params: {
                                    'gameId':gameId,
                                    'goodsType':goodsType,
                                    'tradeType':tradeType
                                }
                            });
                        }
                    }
                }
            });
        }
        return me.store;
    },
    goodsTypeGameGrid: null,
    getGoodsTypeGameGrid: function(){
        var me = this;
        if(Ext.isEmpty(me.goodsTypeGameGrid)){
            me.goodsTypeGameGrid = Ext.widget('gridpanel',{
                header: false,
                columnLines: true,
                store: me.getStore(),
                columns: [{
                    xtype: 'rownumberer'
                },{
                    dataIndex: 'isShowAndroid',
                    text: '是否在Android上显示',
                    flex: .1,
                    sortable: false,
                    align: 'center',
                    renderer: function (value, metaData, record, rowIndex, colIndex, store, view) {
                        var content;
                        if (value) {
                            content = '<div class="container"><div class="leftDiv icons_p_yes"></div>';
                        } else {
                            content = '<div class="container"><div class="leftDiv icons_p_no"></div>';
                        };
                        return content;
                    }
                },{
                    dataIndex: 'isShowIos',
                    text: '是否在iOS上显示',
                    flex: .1,
                    sortable: false,
                    align: 'center',
                    renderer: function(value, metaData, record, rowIndex, colIndex, store, view) {
                        var content;
                        if(value){
                            content = '<div class="container"><div class="leftDiv icons_p_yes"></div>';
                        }else{
                            content = '<div class="container"><div class="leftDiv icons_p_no"></div>';
                        };
                        return content;
                    }
                },{
                    dataIndex: 'gameName',
                    flex: 1,
                    align: 'center',
                    text: '游戏'
                },{
                    dataIndex: 'goodsType',
                    text: '商品类型 ',
                    flex: 1,
                    sortable: false,
                    align: 'center',
                    renderer: function(value, metaData, record, rowIndex, colIndex, store, view) {
                        return DataDictionary.rendererSubmitToDisplay(value,'mainGoodsType');
                    }
                },{
                    dataIndex: 'tradeType',
                    text: '交易类型 ',
                    flex: 1,
                    sortable: false,
                    align: 'center',
                    renderer: function(value, metaData, record, rowIndex, colIndex, store, view) {
                        return DataDictionary.rendererSubmitToDisplay(value,'goodsTradeType');
                    }
                },{
                    dataIndex: 'salable',
                    text: '是否开通出售 ',
                    flex: 1,
                    align: 'center',
                    renderer: function(value, metaData, record, rowIndex, colIndex, store, view) {
                        return DataDictionary.rendererSubmitToDisplay(value,'salable');
                    }
                },{
                    dataIndex: 'tradeDescription',
                    flex: 1,
                    align: 'center',
                    text: '交易说明'
                },{
                    dataIndex: 'editName',
                    flex: .8,
                    align: 'center',
                    text: '操作人'
                }, {
                    dataIndex: 'lastUpdateTime',
                    sortable: false,
                    flex: 1.5,
                    align: 'center',
                    text: '最后修改时间 ',
                    xtype: 'datecolumn',
                    format:'Y-m-d H:i:s'
                }
                ],
                dockedItems: [me.getToolbar(),me.getPagingToolbar()],
                selModel: Ext.create('Ext.selection.CheckboxModel', {
                    allowDeselect: true,
                    mode: 'SINGEL'
                })
            });
        }
        return me.goodsTypeGameGrid;
    },
    initComponent: function() {
        DataDictionary.loadDictDataByGoodsType();
        var me = this;
        Ext.applyIf(me, {
            items: [me.getQueryForm(),me.getGoodsTypeGameGrid()]
        });
        me.callParent(arguments);
    }
});
