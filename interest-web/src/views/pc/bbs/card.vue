<style scoped>
  .content{
    /*padding-right: 88px;*/
    width: 85%;
  }
  .content .content-text{
    border-right: 1px solid #e8eaec;
    padding-right: 16px;
  }
  .content .content-text p{
    word-wrap: break-word;
    word-break: break-all;
    overflow: hidden;
  }
  .content-right{
    width: 15%;
    text-align: center;
    position: absolute;
    right: 0px;
    top: 14px;
    padding-right: 10px;
  }
</style>
<template>
    <div id="mywork">
	    <div class="page-header-main">
	      <div class="box-flex width-80 margin-auto margin-top-2 flex-direction-column flex-justify-center flex-items-center" >
            <div style="width: 100%;margin: 20px 0 20px 0">
                <div class="ivu-card-head" style="background: #eceef2">
                    <p>{{postcard.title}}</p>
                </div>
                <Card>
                    <div class="content-right">
                      <div>
                            <a :href="$store.state.userUrlPre+postcard.userid" target="_blank">
                                <img :src="postcard.headimg" style="width: 25px;height: 25px;border-radius: 100%;">
                                {{postcard.username}}
                            </a>
                      </div>
                      <div style="background: #5cadff;text-align: center;color: #fff;border-radius: 10px;">
                        楼主
                      </div>
                    </div>
                    <div class="content">
                        <div class="content-text">
                          <p>{{postcard.content}}</p>
                          <span>
                              <Icon type="ios-time"></Icon>
                              {{postcard.createtime}}
                          </span>
                        </div>
                    </div>
                </Card>

                <Card v-for="(item,index) in replyCardList" :key="index">
                    <div class="content-right">
                        <a :href="$store.state.userUrlPre+item.userId" target="_blank">
                            <img :src="item.headImg" style="width: 25px;height: 25px;border-radius: 100%;">
                            {{item.userName}}
                        </a>
                    </div>
                    <div class="content">
                      <div class="content-text">
                        <p>{{item.content}}</p>
                        <span>
                            <Icon type="ios-time"></Icon>
                            {{item.createTime}}
                        </span>
                      </div>
                    </div>
                </Card>
                <div style="margin-top: 20px">
                    <Page :total="total" :page-size="pageInfo.pageSize" show-elevator show-total @on-change="e=>{pageSearch(e)}"></Page>
                </div>
            </div>

            <div class="box-flex width-100 margin-auto margin-top-2 border-top border-color-bfbfbf"></div>

            <div class="box-flex margin-auto margin-top-2 flex-direction-column flex-justify-center flex-items-center" style="width: 100%">
                <div class=" width-100 flex-direction-row">
                  <div class="box-flex flex-1 padding-all-5x">
                    <span><Icon type="edit"></Icon>发表回复</span>
                  </div>
                  <div class="box-flex flex-6 width-100 padding-all-5x">
                    <Input v-model="textarea" type="textarea" :rows="6" placeholder="内容" />
                  </div>
                </div>
                <div class="box-flex width-100 margin-top-2 flex-items-flex-end flex-justify-flex-end margin-bottom-3">
                  <Button type="primary" @click="sendCard()">发表</Button>
                </div>
              </div>
	      </div>
	    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      total: 0,
      pageInfo: {
        page: 0,
        pageSize: 20
      },
      textarea: "",
      postcardid: null,
      postcard: {
        id: "",
        username: "",
        title: "",
        content: "",
        interestid: "",
        createtime: "",
        headimg: "",
        githuburl: "",
        userid: ""
      },
      replyCardList: []
    };
  },
  mounted() {
    this.postcardid = this.$route.params.id;
    this.axios({
      method: "get",
      url: "/interest/bbs/public/postcards/postcard",
      params: {
        id: this.postcardid
      }
    })
      .then(
        function(response) {
          this.postcardSet(response.data.data);
        }.bind(this)
      )
      .catch(
        function(error) {
          alter(error);
        }.bind(this)
      );
    this.replyCardListGet({
      pageInfo: this.pageInfo,
      postcardid: this.postcardid
    });
  },
  methods: {
    replyCardListGet(e) {
      this.axios({
        method: "get",
        url: "/interest/bbs/public/reply-cards",
        params: {
          page: e.pageInfo.page,
          pageSize: e.pageInfo.pageSize,
          postCardId: e.postcardid
        }
      })
        .then(
          function(response) {
            this.replyCardList = response.data.data.data;
            this.listDateSet(this.replyCardList);
            this.total = response.data.data.totalCount;
          }.bind(this)
        )
        .catch(function(error) {
          alert(error);
        });
    },

    listDateSet(e) {
      for (var i = e.length - 1; i >= 0; i--) {
        e[i].createTime = this.dateGet(e[i].createTime);
      }
    },
    postcardSet(e) {
      console.log(e);
      this.postcard.id = e.id;
      this.postcard.username = e.userName;
      this.postcard.title = e.title;
      this.postcard.content = e.content;
      this.postcard.interestid = e.interestId;
      this.postcard.createtime = this.dateGet(e.createTime);
      this.postcard.headimg = e.headImg;
      this.postcard.userid = e.userId;
    },
    pageSearch(e) {
      this.pageInfo.page = e - 1;
      this.replyCardListGet({
        pageInfo: this.pageInfo,
        postcardid: this.postcardid
      });
    },
    sendCard() {
      if (this.textarea != null && this.textarea != "") {
        if (
          this.axios.defaults.headers.common["Authorization"] != null &&
          this.axios.defaults.headers.common["Authorization"] != ""
        ) {
          this.axios({
            method: "post",
            url: "/interest/bbs/reply-cards/reply-card",
            data: {
              postCardId: this.postcardid,
              content: this.textarea
            }
          })
            .then(
              function(response) {
                this.$Message.info("回复成功");
                this.textarea = "";
                this.replyCardListGet({
                  pageInfo: this.pageInfo,
                  postcardid: this.postcardid
                });
              }.bind(this)
            )
            .catch(
              function(error) {
                this.$Message.error("请重新登录");
              }.bind(this)
            );
        } else {
          this.$Message.error("登录后，才能回复！");
        }
      } else {
        this.$Message.error("请填写回复内容");
      }
    }
  }
};
</script>
