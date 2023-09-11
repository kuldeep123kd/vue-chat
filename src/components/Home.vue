<template>
  <div class="message-board-container">
      <div class="message-board-content">
        <aside class="users-list-aside" :class="{'users-list-aside-mob-active':user_list_on_mobile}">
            <div class="users-list-mobile-screen" :class="{'users-list-mobile-screen-active':user_list_on_mobile}" @click="showUserListOnMobile">
                <img class="users-list-mobile-screen-arrow" src="/img/left_arrow.png" width="20" height="20" />
            </div>
            <div v-if="user_list_on_mobile" class="users-list-mobile-screen-backdrop">

            </div>
            <h1 class="users-list-chat">Chat</h1>
            <div class="users-list-container">
                <h1 class="users-list-heading">Users</h1>
                <div class="users-list-data-container">
                    <div v-for="(user, user_index) in user_list" @click="selectUser(user)" :key="user_index+'user'" class="users-list-data">
                        <avatar
                            :username="user.user_name"
                            :src="''"
                            :size="48"
                            :rounded="true"
                            color="#fff">
                        </avatar>
                        <h2 class="user-name">
                            {{user.user_name}}
                        </h2>
                    </div>
                </div>
            </div>
            <div class="logout-btn" @click="logoutUser">
                <img src="/img/logout_vector.svg" width="20" height="20" />
                <span class="logout-btn-text">
                    Log out
                </span>
            </div>
        </aside>
        <main class="main">
            <div v-if="selected_user" class="main-container">
                <div class="main-content-header">
                    <div class="main-content-user">
                        <avatar
                            :username="selected_user.user_name"
                            :src="''"
                            :size="48"
                            :rounded="true"
                            color="#fff">
                        </avatar>
                        <h2 class="user-name">
                            {{selected_user.user_name}}
                        </h2>
                    </div>
                    <div class="main-content-user-filter">
                        <ul class="main-content-user-filter-list">
                            <li :class="{'active': chat_filter_option_selected == 'all'}" @click="chatFilterOption('all')">
                                All
                            </li>
                            <li :class="{'active': chat_filter_option_selected == 'today'}" @click="chatFilterOption('today')">
                                Today
                            </li>
                            <li :class="{'active': chat_filter_option_selected == 'last_7_days'}" @click="chatFilterOption('last_7_days')">
                                Last 7 days
                            </li>
                        </ul>
                    </div>
                </div>
                <div class="main-content-chat-area">
                    <div class="main-content-chat-area-messages-container">
                        <div v-if="selected_user.users_chat.length" class="w-100 align-self-end">
                            <div v-for="(chat_data, chat_data_index) in selected_user.users_chat" :key="chat_data_index+'chat_data_index'" class="main-content-chat-area-messages">
                                <pre class="" :class="{'main-content-chat-area-messages-text-by-author': chat_data.user_name == logged_user.user_name,'main-content-chat-area-messages-text-by-user': chat_data.user_name != logged_user.user_name}">
                                    {{chat_data.message}}
                                </pre>
                                <p class="" :class="{'main-content-chat-area-messages-by-author': chat_data.user_name == logged_user.user_name,'main-content-chat-area-messages-by-user': chat_data.user_name != logged_user.user_name}">
                                    {{chat_data.user_name}} - {{moment(chat_data.date).fromNow()}}
                                </p>
                            </div>
                        </div>
                        <div v-else class="text-center no-messages-found">
                            <span>Please send your first message</span>
                        </div>
                    </div>
                    <div class="main-content-chat-area-input">
                        <textarea class="main-content-chat-area-input-field" ref="chatInput" type="text" placeholder="Write a message..." />
                        <img class="main-content-chat-area-input-btn" @click="submitChat" src="/img/message_input.svg" width="36" height="36" />
                    </div>
                </div>
            </div>
        </main>
      </div>
  </div>
</template>

<script >
import moment from 'moment-timezone';
export default {
    name: 'HelloWorld',
    props: {
        msg: String
    },
    data() {
        return {
            chat_filter_option_selected: 'all',
            user_list_on_mobile: false,
            user_list: [],
            selected_user: null,
            stored_selected_user: null,
            stored_users_data: null,
            current_user_chat: []
        }
    },
    methods: {
        moment(date) {
            return moment(date);
        },
        chatFilterOption(option) {
            if(option == 'today') {
                this.chat_filter_option_selected = 'today';
                this.filterDataByDate('today');
            } else if(option == 'last_7_days') {
                this.chat_filter_option_selected = 'last_7_days'
                this.filterDataByDate('last_7_days');
            } else {
                this.chat_filter_option_selected = 'all'
                this.filterDataByDate('all');
            } 
        },
        showUserListOnMobile() {
            this.user_list_on_mobile = !this.user_list_on_mobile
        },
        selectUser(user) {
            this.selected_user = user;
            this.stored_selected_user = JSON.parse(JSON.stringify(user))
            this.chatFilterOption('all')
            // let user_index = this.user_list.findIndex(user => user.name = )
        },
        logoutUser() {
            let user_index = this.stored_users_data.findIndex(user => user.user_name == this.logged_user.user_name);
            if(user_index >= 0) {
                this.stored_users_data.splice(user_index, 1)
                localStorage.setItem('chat_users_data', JSON.stringify(this.stored_users_data));
            }
            sessionStorage.removeItem('user_login_token');
            this.$router.push('/login')
        },
        submitChat() {
            if(this.$refs.chatInput.value.length) {
                this.selected_user.users_chat.push({
                    date: new Date(),
                    message: this.$refs.chatInput.value,
                    user_name: this.logged_user.user_name
                })
                this.stored_selected_user.users_chat.push({
                    date: new Date(),
                    message: this.$refs.chatInput.value,
                    user_name: this.logged_user.user_name
                })
                let user_index = this.stored_users_data.findIndex(user => user.user_name == this.selected_user.user_name);
                this.stored_users_data[user_index].users_chat = this.stored_selected_user.users_chat;
                localStorage.setItem('chat_users_data', JSON.stringify(this.stored_users_data));
                this.$refs.chatInput.value = ''
            }
        },
        filterDataByDate(type) {
            let temp_data = JSON.parse(JSON.stringify(this.stored_selected_user))
            const filteredData = temp_data.users_chat.filter(item => {
                const itemDate = new Date(item.date);
                if(type == 'today') {
                    return itemDate.getDate() === new Date().getDate();
                } else if(type == 'last_7_days') {
                    let targetDate = new Date()
                    const oneWeekAgo = new Date(targetDate);
                    oneWeekAgo.setDate(targetDate.getDate() - 7);
                    return itemDate >= oneWeekAgo && itemDate <= targetDate;
                } else {
                    return true;
                }
            });
            // return filteredData;
            this.selected_user.users_chat = filteredData
            this.$forceUpdate()
        },
        loadChatData() {
            this.stored_users_data = JSON.parse(localStorage.getItem('chat_users_data'))
            if(this.stored_users_data) {
                this.user_list = JSON.parse(JSON.stringify(this.stored_users_data))
            }
            if(this.user_list.length) {
                this.selected_user = this.user_list[0];
                this.stored_selected_user = JSON.parse(JSON.stringify(this.user_list[0]));
            }
        },
        loadChatDataOnInterval() {
            this.stored_users_data = JSON.parse(localStorage.getItem('chat_users_data'))
            if(this.stored_users_data) {
                this.user_list = JSON.parse(JSON.stringify(this.stored_users_data))
            }
            if(this.user_list.length) {
                let user_index = this.stored_users_data.findIndex(user => user.user_name == this.selected_user.user_name);
                this.stored_selected_user.users_chat = JSON.parse(JSON.stringify(this.stored_users_data[user_index].users_chat));
                this.selected_user.users_chat = JSON.parse(JSON.stringify(this.stored_users_data[user_index].users_chat));
            }
        }
    },
    mounted() {
        let login_token = sessionStorage.getItem('user_login_token');
        // console.log(login_token)
        if(login_token == null || login_token == undefined) {
            this.$router.push('/login')
        }
        if(login_token) {
            this.loadChatData();
            this.chatIntervalID = setInterval(()=> {
                // console.log('Time Interval Check')
                this.loadChatDataOnInterval();
            }, 1000);
        }
    },
    computed: {
        logged_user() {
            let login_token = sessionStorage.getItem('user_login_token');
            let decoded_login_token = ''
            if(login_token) {
                decoded_login_token = atob(login_token)
            }
            return {
                user_name: decoded_login_token
            }
        }
    },
    beforeDestroy() {
        clearInterval(this.chatIntervalID);
    },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
    .message-board-container {
        height: 100%;
        .message-board-content {
            height: 100%;
            display: flex;
            .users-list-aside {
                position: fixed;
                left: -78.8px;
                top: 0;
                bottom: 0;
                width: 48px;
                min-width: 48px;
                padding: 15px 15px 25px 15px;
                border-right: 1px solid #F3F3F3;
                background-color: #fff;
                display: flex;
                flex-direction: column;
                transition: all 0.3s linear;
                z-index: 1;
                &.users-list-aside-mob-active {
                    left: 0;
                }
                @media screen and (min-width: 415px) {
                    left: 0;
                }
                @media screen and (min-width: 992px) {
                    width: 250px;
                    min-width: 250px;
                    padding: 41px 41px 25px 41px;
                }
                .users-list-mobile-screen {
                    padding: 5px;
                    background: #ececec;
                    position: fixed;
                    top: 10px;
                    left: 0;
                    display: flex;
                    align-items: center;
                    justify-content: center;
                    z-index: 1;
                    border-top-right-radius: 10px;
                    border-bottom-right-radius: 10px;
                    cursor: pointer;
                    transition: all 0.3s linear;
                    &.users-list-mobile-screen-active {
                        left: 78.8px;
                    }
                    @media screen and (min-width: 415px) {
                        display: none;
                    }
                    .users-list-mobile-screen-arrow{

                    }
                }
                .users-list-mobile-screen-backdrop {
                    position: fixed;
                    left: 78.8px;
                    top: 0;
                    bottom: 0;
                    right: 0;
                    background: rgba(0,0,0,0.3);
                    cursor: not-allowed;
                    display: block;
                    @media screen and (min-width: 415px) {
                        display: none;
                    }
                }
                .users-list-chat {
                    font-size: 20px;
                    font-weight: 500;
                    text-align: left;
                    color: #161616;
                    @media screen and (min-width: 992px) {
                        font-size: 36px;
                    }
                }
                .users-list-container {
                    height: 100%;
                    overflow-y: auto;
                    .users-list-heading {
                        font-size: 14px;
                        font-weight: 500;
                        text-align: left;
                        color: #161616;
                        @media screen and (min-width: 992px) {
                            font-size: 18px;
                        }
                    }
                    .users-list-data-container {
                        .users-list-data {
                            display: flex;
                            align-items: center;
                            margin: 15px 0;
                            cursor: pointer;
                            // .user-image {
                            //     display: block;
                            //     width: 48px;
                            //     height: 48px;
                            //     border-radius: 100px;
                            //     background: #6D42D8;
                            // }
                            .user-name {
                                font-size: 16px;
                                font-weight: 600;
                                text-align: left;
                                padding-left: 15px;
                                margin: 0;
                                display: none;
                                text-transform: capitalize;
                                @media screen and (min-width: 992px) {
                                    display: block;
                                }
                            }
                        }
                    }
                }
                .logout-btn {
                    display: flex;
                    align-items: center;
                    margin-top: 10px;
                    cursor: pointer;
                    img {
                        margin: 0 auto;
                        @media screen and (min-width: 992px) {
                            margin: 0;
                        }
                    }
                    .logout-btn-text {
                        font-size: 16px;
                        font-weight: 500;
                        text-align: left;
                        color: #6D42D8;
                        padding-left: 10px;
                        display: none;
                        @media screen and (min-width: 992px) {
                            display: block;
                        }
                    }
                }
            }
            .main {
                margin-left: 0px;
                width: 100%;
                transition: all 0.3s linear;
                @media screen and (min-width: 415px) {
                    margin-left: 78.8px;
                }
                @media screen and (min-width: 992px) {
                    margin-left: 332.8px;
                }
                .main-container {
                    height: 100%;
                    display: flex;
                    flex-direction: column;
                    .main-content-header {
                        display: flex;
                        align-items: center;
                        justify-content: space-between;
                        flex-wrap: wrap;
                        padding: 20px 30px 30px 30px;
                        position: sticky;
                        top: 0px;
                        background: #fff;
                        .main-content-user {
                            display: flex;
                            align-items: center;
                            margin-top: 10px;
                            // .user-image {
                            //     display: block;
                            //     width: 48px;
                            //     min-width: 48px;
                            //     height: 48px;
                            //     border-radius: 100px;
                            //     background: #6D42D8;
                            // }
                            .user-name {
                                font-size: 16px;
                                font-weight: 600;
                                text-align: left;
                                padding-left: 15px;
                                margin: 0;
                                text-transform: capitalize;
                            }
                        }
                        .main-content-user-filter {
                            background: #F9F9F9;
                            padding: 5px;
                            margin-top: 10px;
                            .main-content-user-filter-list {
                                display: flex;
                                align-items: center;
                                padding: 0;
                                margin: 0;
                                li {
                                    cursor: pointer;
                                    min-width: auto;
                                    list-style: none;
                                    border-radius: 6px;
                                    color: #161616;
                                    background: #F9F9F9;
                                    padding: 8px 20px;
                                    &.active {
                                        color: #fff;
                                        background: #6D42D8;
                                    }
                                    @media screen and (min-width: 510px) {
                                        min-width: 78px;
                                    }
                                }
                            }
                        }
                    }
                    .main-content-chat-area {
                        background: #F8F8F9;
                        height: 100%;
                        width: 100%;
                        margin-bottom: 90px;
                        overflow-y: auto;
                        position: relative;
                        display: flex;
                        flex-direction: column-reverse;
                        .main-content-chat-area-messages-container {
                            padding: 0px 15px 0px 30px;
                            .main-content-chat-area-messages {
                                padding: 10px 0;
                                .main-content-chat-area-messages-text-by-user {
                                    border: 1px solid #E5EBEF;
                                    background: linear-gradient(0deg, #E5EBEF, #E5EBEF),linear-gradient(0deg, #FBFBFD, #FBFBFD);
                                    padding: 12px 16px 12px 16px;
                                    border-radius: 0px 8px 8px 8px;
                                    color: #161616;
                                    font-size: 14px;
                                    font-weight: 500;
                                    text-align: left;
                                    margin: 0 auto 0 0;
                                    white-space: pre-line;
                                    max-width: calc(100% - 45px);
                                    width: fit-content;
                                }
                                .main-content-chat-area-messages-text-by-author {
                                    padding: 12px 16px 12px 16px;
                                    border-radius: 8px 0px 8px 8px;
                                    background: #6D42D8;
                                    font-size: 14px;
                                    font-weight: 500;
                                    text-align: right;
                                    margin: 0 0 0 auto;
                                    white-space: pre-line;
                                    max-width: calc(100% - 45px);
                                    width: fit-content;
                                    color: #fff;
                                }
                                .main-content-chat-area-messages-by-user,
                                .main-content-chat-area-messages-by-author {
                                    font-size: 12px;
                                    font-weight: 400;
                                    color: #869AA9;
                                    text-transform: capitalize;
                                }
                                .main-content-chat-area-messages-by-user {
                                    text-align: left;
                                }
                                .main-content-chat-area-messages-by-author {
                                    text-align: right;
                                }
                            }
                            .no-messages-found {
                                position: absolute;
                                bottom: 15px;
                                left: 50%;
                                transform: translateX(-50%);
                            }
                        }
                        .main-content-chat-area-input{
                            background: #FFFFFF;
                            padding: 15px 15px;
                            display: flex;
                            position: fixed;
                            bottom: 0;
                            right: 0;
                            left: 0;
                            transition: all 0.3s linear;
                            @media screen and (min-width: 415px) {
                                left: 78.8px;
                            }
                            @media screen and (min-width: 992px) {
                                left: 332.8px;
                            }
                            .main-content-chat-area-input-field {
                                padding: 12px 50px 12px 16px;
                                outline: 0 none;
                                border: 0 none;
                                width: 100%;
                                resize: none;
                                &::-webkit-scrollbar {
                                    display: none;
                                }
                            }
                            .main-content-chat-area-input-btn{
                                cursor: pointer;
                                border-radius: 100px;
                                position: absolute;
                                right: 20px;
                                top: 20px;
                            }
                        }
                    }
                }
            }
        }
    }
</style>
