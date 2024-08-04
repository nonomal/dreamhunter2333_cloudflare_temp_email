<script setup>
import { ref } from 'vue'
import { useI18n } from 'vue-i18n'
import { startRegistration } from '@simplewebauthn/browser';

import { useGlobalState } from '../../store'
import { api } from '../../api'

const { userJwt, userSettings, } = useGlobalState()
const message = useMessage()

const showLogout = ref(false)

const { t } = useI18n({
    messages: {
        en: {
            logout: 'Logout',
            logoutConfirm: 'Are you sure you want to logout?',
            passordTip: 'The server will only receive the hash value of the password, and will not receive the plaintext password, so it cannot view or retrieve your password. If the administrator enables email verification, you can reset the password in incognito mode',
            createPasskey: 'Create Passkey',
            viewPasskeys: 'View Passkeys',
            passkeyCreated: 'Passkey created successfully',
        },
        zh: {
            logout: '退出登录',
            logoutConfirm: '确定要退出登录吗？',
            passordTip: '服务器只会接收到密码的哈希值，不会接收到明文密码，因此无法查看或者找回您的密码, 如果管理员启用了邮件验证您可以在无痕模式重置密码',
            createPasskey: '创建 Passkey',
            viewPasskeys: '查看 Passkeys',
            passkeyCreated: 'Passkey 创建成功',
        }
    }
});


const logout = async () => {
    userJwt.value = '';
    location.reload()
}

const createPasskey = async () => {
    try {
        const options = await api.fetch(`/user_api/passkey/register_request`, {
            method: 'POST',
            body: JSON.stringify({
                domain: location.hostname,
            })
        })
        const credential = await startRegistration(options)

        // Send the result to the server and return the promise.
        await api.fetch(`/user_api/passkey/register_response`, {
            method: 'POST',
            body: JSON.stringify({
                origin: location.origin,
                passkey_name: (
                    (window.navigator.userAgentData?.platform || "Unknown")
                    + ": " + Math.random().toString(36).substring(7)
                ),
                credential
            })
        })
        message.success(t('passkeyCreated'));
    } catch (e) {
        console.error(e)
        message.error(e.message)
    }
}
</script>

<template>
    <div class="center" v-if="userSettings.user_email">
        <n-card :bordered="false" embedded>
            <n-button secondary block strong>
                {{ t('viewPasskeys') }}
            </n-button>
            <n-button @click="createPasskey" type="primary" secondary block strong>
                {{ t('createPasskey') }}
            </n-button>
            <n-alert :show-icon="false" :bordered="false">
                <span>
                    {{ t('passordTip') }}
                </span>
            </n-alert>
            <n-button @click="showLogout = true" secondary block strong>
                {{ t('logout') }}
            </n-button>
        </n-card>
        <n-modal v-model:show="showLogout" preset="dialog" :title="t('logout')">
            <p>{{ t('logoutConfirm') }}</p>
            <template #action>
                <n-button :loading="loading" @click="logout" size="small" tertiary type="primary">
                    {{ t('logout') }}
                </n-button>
            </template>
        </n-modal>
    </div>
</template>

<style scoped>
.center {
    display: flex;
    justify-content: center;
}


.n-card {
    max-width: 800px;
    text-align: left;
}

.n-button {
    margin-top: 10px;
    margin-bottom: 10px;
}
</style>
