<template>
  <div class="page">
    <div class="main-content">
      <div class="header">
        <p class="title">学术家族树</p>
      </div>
      <div class="body">
        <textarea class="body-text" placeholder="请输入学术关系(导师应位于第一行输入)...." id="data"></textarea>
        <button class="create-btn" @click="processdata">一键生成</button>
      </div>
      <div class="footer"></div>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      son: new Map(),//记录子节点
      info: new Map(),//最高学历
      mark: new Map(), //判断是否有父节点
      data: [],
      ID: 1
    };
  },
  methods: {
    processdata(){
        let txt=document.getElementById("data");
        var data=txt.value;
        var str=data.split("\n");//将数据以换行符为间隔隔开
        let list=["博士生","硕士生","本科生"];//关键词
        let grad=new Map();
        (grad["导师"] = 0),
        (grad["博士生"] = 1),
        (grad["硕士生"] = 2),
        (grad["本科生"] = 3);
        let checkinput=["级博士生","级硕士生","级本科生"];
        var ID=0;
        var that=this;
        var check=0;
        for(var i=1;i<str.length;i++)//异常处理
        {
            for(var value of checkinput){
                if(str[i].indexOf(value)!=-1)
                check=1;
            }
            if(!check){
                alert("第"+(i+1)+"行输入错误，请检查并重新输入！（输入规范：**级**生：name）");
                return ;
            }
            check=0;
        }
        for(var i=0;i<str.length;){
            var j;
            for(j=i+1;j<str.length;j++){
                if(str[j]=="\n")
                break;
            }
            var m=str[i].indexOf("导师");
            if(m==-1){
            alert("输入错误，请重新输入！(是否输入了导师？）");
            return ;
        }
            let master=str[i].substring(m+3);
            that.son[master]=[];//使用二维map记录子节点
            this.info[master]="导师";//导师最高学历为导师
            var k;
            for(k=i+1;k<j;k++){
                var grade,idx,year;
                for(var value of list){
                    idx=str[k].indexOf(value);
                    if(idx!=-1){
                        year=str[k].substring(0,idx);
                        grade=str[k].substring(idx,idx+3);
                        if(that.son[year]==null)
                        that.son[master].push(year);
                        if(that.son[year]==null)
                        that.son[year]=[];
                        if(that.son[year+grade]==null)
                        {
                        that.son[year].push(year+grade);
                        that.son[year+grade]=[];
                        }
                        that.info[grade]=value;
                        that.info[year+grade]=value;
                        that.mark[year+grade]=1;
                        that.mark[year]=1;
                        break;
                    }
                    
                }
                var stuname=str[k].substring(idx+4).split("、");
                for(var value of stuname){
                    if(that.info[value]==null||grad[that.info[value]]>grad[grade])
                    that.info[value]=grade;
                    that.son[year+grade].push(value);
                    that.mark[value]=1;
                }
            }
                i=j+1;
        }
            for(var key in that.info)//查找根节点
            {
                if(that.mark[key]==null){
                    that.data.push(that.formatData(key,-1));
                }
            }
           this.$router.push({
                path: "/tree",
                query: {
                 data: JSON.stringify(that.data)
                }
      });
    },
    formatData(rt, fa) {
      let now = {};
      now.name = rt;
      now.id = this.ID;
      this.ID += 1;
      now.lv = this.info[rt];
      now.children = [];
      let arr = this.son[rt];
      if (arr == null) return now;
      for (var i = 0; i < arr.length; i++) {
        now.children.push(this.formatData(arr[i], rt));
      }
      if(rt.indexOf("生")!=-1){
          let tmp=rt.substring(rt.indexOf("级")+1);
          now.name=tmp;
      }
      return now;
    }
  }
};
</script>