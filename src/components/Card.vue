<template>
        <md-layout md-flex-xsmall="100" md-flex-small="50" md-flex-medium="33" md-gutter>
            <md-card class="card-task">
                <md-card-area md-inset>
                    <md-card-media md-ratio="16:9">
                        <img v-bind:src="item.imgUrl" alt="任务图片">
                    </md-card-media>

                    <md-card-header>
                    <h2 class="md-title">{{item.title}}</h2>
                    <div class="md-subhead description">
                        {{item.description}}
                    </div>
                    <div class="md-subhead">
                        <md-icon>location_on</md-icon>
                        <span>{{item.todoList[0].todoAddress}}</span>
                    </div>
                    </md-card-header>
                </md-card-area>
                

                <md-card-expand>
                    <md-card-actions>
                        <md-button id="starButton" @click.native="likeUp(item.id)" class="md-raised md-default starButton"><md-icon>thumb_up</md-icon> ({{likes}})</md-button>
                        <md-button id="shareButton" @click.native="handleShare(item.id)" class="md-icon-button md-raised md-default shareButton"><md-icon>share</md-icon></md-button>
                        <span style="flex: 1"></span>
                        <md-button class="md-icon-button" md-expand-trigger>
                            <md-icon>keyboard_arrow_down</md-icon>
                        </md-button>
                    </md-card-actions>
                    <md-card-content>
                        <md-card-area>
                            <h3 class="md-subheading">{{ $t("Card.timeHeading") }}</h3>
                            <div class="card-reservation">
                            <md-icon>access_time</md-icon>
                                <p class="activity_time">{{ $t("Card.date") }}：{{item.date}} {{ $t("Card.time") }}: {{item.todoList[0].todoDate}}</p>
                            </div>
                        </md-card-area>
                        <md-card-actions>
                            <md-button @click.native="goDetail(item.id)" id="detailButton" class="md-raised">{{ $t("Card.detail") }}</md-button>
                            <md-button @click.native="joinNow(item.id)" id="joinButton" class="md-raised md-accent" :disabled="joinButtonDisable">{{joinButtonText}}</md-button>
                        </md-card-actions>
                    </md-card-content>
                </md-card-expand>
            </md-card>
        </md-layout>
</template>

<script>
    import AV from 'leancloud-storage';

    export default {
        name: 'Card',
        props: ['item','index'],
    
        data () {
            
            return {
                likes : this.item.likes,
                activityItem: this.item,
                joinButtonText: this.item.joinButtonText,
                joinButtonDisable: this.item.joinButtonDisable
            }
        },
        methods: {
            likeUp: function(activityID) {
                //console.log('like UP!', activityID);
                let query = new AV.Query('Activity');
                query.get(activityID).then((activity)=>{
                    activity.increment('likes', 1);
                    return activity.save(null, {
                        fetchWhenSave: true
                    });
                }, (error)=>{
                    console.log("发生错误！错误代码：", error);
                }).then((activity)=>{
                    console.log("点赞成功！");
                    this.likes = activity.get("likes");
                }, (error)=>{
                    console.log("点赞出错！error: ", error);
                })
            },
            handleShare: function(activityID) {
                console.log('share button clicked!', activityID);
            },
            joinNow: function(activityID){
                console.log('joinNow clicked!', activityID);
                let user = AV.User.current();
                console.log(this.item.AVObj);
                let activity = new AV.Query("Activity");
                activity.get(this.activityItem.id).then((item)=>{
                    item.get('joinUsers').push(user);
                    item.save();
                },(error)=>{
                    console.log("获取该任务信息出错")
                }).then((msg)=>{
                    console.log("activity 存储完成，msg: ", msg);
                    console.log("接下来存储user方的joinedActivities");
                    user.get("joinedActivities").push(this.item.AVObj);
                    user.save();
                }, (error)=>{
                    console.log("activity.joinuser存储失败，error: ", error);
                }).then((msg)=>{
                    console.log("参与成功！");
                    this.joinButtonText = "Joined Task";
                    this.joinButtonDisable = true;
                }, (error)=>{
                    console.log("参与活动失败，error: ", error);
                });
                // user.get("joinedActivities").push(this.item.AVObj);
                // user.save().then((msg)=>{
                //     console.log("参与成功！");
                //     this.joinButtonText = "Joined Task";
                //     this.joinButtonDisable = true;
                // }, (error)=>{
                //     console.log("参与活动失败，error: ", error);
                // })
            },
            goDetail: function(activityID){
                this.$router.push({
                    path: "activity_detail/"+ activityID
                })
            }
        }
    }
</script>

<style lang="scss">
    .description{
        min-width: 300px;
        max-width: 300px;
    }

    .md-card-header {
        .md-title {
            color: rgba(0, 0, 0, .87);
        }
        text-align: left;
    }

    .md-card-expand {
        .md-subheading {
            text-align: center;
        }
        .card-reservation {
            margin-top: 8px;
            display: flex;
            align-items: center;
            justify-content: space-around;

            .md-icon {
            margin: 8px;
            color: rgba(#000, .54) !important;
            }

            .md-button {
            border-radius: 2px !important;
            }
        }
    }

    #starButton {
        margin: 6px 8px;
    }

    #joinButton, #detailButton {
        margin: 10px auto -2px auto;
    }


</style>