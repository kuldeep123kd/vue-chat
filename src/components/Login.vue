<template>
    <div class="login-container">
        <div>
            <div class="comp-logo">
                <span class="comp-logo-img"></span>
                Logo
            </div>
            <div class="login-content">
                <div class="login-content-main">
                    <h1 class="login-text">
                        Log in
                    </h1>
                    <div class="login-input-container">
                        <div>
                            <label class="user-name-label" for="user_name">
                                User name
                            </label>
                            <input class="user-name-input" @input="onUserNameInput" type="text" name="user_name" v-model="user_name" placeholder="" />
                            <span v-if="username_has_error" class="user-name-error">
                                Please enter user name
                            </span>
                            <div class="login-btn-container">
                                <button class="login-btn" type="button" @click="loginSubmit" >Log In</button>
                            </div>
                        </div>
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
            user_name: '',
            username_has_error: false
        }
    },
    methods: {
        onUserNameInput(event) {
            if(event.target.value.length) {
                this.username_has_error = false
            }
        },
        loginSubmit() {
            if(this.user_name.length > 0) {
                this.username_has_error = false
            } else {
                this.username_has_error = true
            }
            if(!this.username_has_error) {
                let login_token = btoa(this.user_name)
                let users_array = [];
                let stored_users_data = JSON.parse(localStorage.getItem('chat_users_data'))
                if(stored_users_data && stored_users_data.length) {
                    users_array = stored_users_data
                }
                let user_index = users_array.findIndex(user => user.user_name == this.user_name);
                if(user_index == -1) {
                    users_array.push({
                        user_name: this.user_name.trim(),
                        created_at: new Date(),
                        user_image: '',
                        users_chat: []
                    });
                }
                localStorage.setItem('chat_users_data', JSON.stringify(users_array));
                localStorage.setItem('user_login_token', login_token);
                this.$router.push('/')
            }
            
        }
    },
    mounted() {
    }
}
</script>

<style lang="scss" scoped>
    .login-container {
        top: 50%;
        position: relative;
        transform: translateY(-50%);
        padding: 0 15px;
        .comp-logo{
            font-size: 39px;
            font-weight: 700;
            line-height: 58px;
            letter-spacing: 0px;
            text-align: center;
            margin-top: 0;
            color: #171725;
            display: flex;
            align-items: center;
            justify-content: center;
            margin-bottom: 20px;
            .comp-logo-img {
                background: #6D42D8;
                width: 28px;
                height: 28px;
                border-radius: 100px;
                margin-right: 10px;
            }
        }
        .login-content {
            max-width: 568px;
            min-height: 335px;
            margin: auto;
            box-shadow: 0px 9px 24px 0px rgba(0, 0, 0, 0.18);
            border-radius: 14px;
            .login-content-main {
                padding: 50px 15px;
                .login-text {
                    font-size: 1.875rem;
                    font-weight: 600;
                    color: #171725;
                    margin: 0;
                }
                .login-input-container {
                    max-width: 402px;
                    margin: 0 auto;
                    .user-name-label {
                        color: rgba(150, 154, 184, 1);
                        display: block;
                        text-align: left;
                        margin-block: .625rem;
                    }
                    .user-name-input {
                        border: 1.6px solid rgba(224, 226, 233, 1);
                        border-radius: 8px;
                        width: calc(100% - 35.2px);
                        outline: 0 none;
                        padding: 0.875rem 1rem;
                    }
                    .user-name-error {
                        color: #ff3333;
                        font-size: 1rem;
                        text-align: left;
                        display: block;
                    }
                    .login-btn-container {
                        margin-top: 1.25rem;
                        .login-btn {
                            background: rgba(109, 66, 216, 1);
                            width: 100%;
                            padding: 0.75rem 1rem;
                            text-align: center;
                            outline: 0 none;
                            border-radius: 8px;
                            border: 0 none;
                            font-size: 15px;
                            font-weight: 600;
                            color: #fff;
                            cursor: pointer;
                        }
                    }
                }
            }
        }
    }
</style>