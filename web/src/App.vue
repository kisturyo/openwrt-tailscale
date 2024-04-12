<template>
    <div id="main">
        <!--  -->
        <h2>Tailscale</h2>
        <div class="cbi-map-descr">
            The easiest, most secure way to use WireGuard and 2FA.
        </div>
        <!--  -->
        <div class="cbi-section">
            <div class="cbi-section-node">
                <div class="cbi-value cbi-value-last">
                    <label class="cbi-value-title">Enable</label>
                    <div class="cbi-value-field">
                        <div class="cbi-checkbox">
                            <input name="enabled" type="checkbox" :value="false" v-model="config.enabled">
                            <label></label>
                        </div>
                    </div>
                </div>
                <div class="cbi-value cbi-value-last">
                    <label class="cbi-value-title">Tailscale</label>
                    <div class="cbi-value-field">
                        <a v-if="loading">Loading...</a>
                        <template v-else>
                            <a v-if="status?.BackendState" style="color:green"> {{ status?.BackendState }}</a>
                            <a v-else style="color:red">NOT RUNNING</a>
                        </template>
                    </div>
                </div>
                <div class="cbi-value cbi-value-last" v-if="status?.Self?.ID">
                    <label class="cbi-value-title">Current node</label>
                    <div class="cbi-value-field">
                        <a>
                            {{ status?.Self?.ID }}
                            <template v-if="status?.Self?.HostName">
                                （{{ status.Self.HostName }}）
                            </template>
                        </a>
                    </div>
                </div>
                <div class="cbi-value cbi-value-last" v-if="status?.AuthURL">
                    <label class="cbi-value-title">Verify to binding</label>
                    <div class="cbi-value-field">
                        <a :href="status.AuthURL" target="_blank">{{ status.AuthURL }}</a>
                    </div>
                </div>
                <div class="cbi-value cbi-value-last" v-if="user?.DisplayName">
                    <label class="cbi-value-title">Bound devices</label>
                    <div class="cbi-value-field">
                        <a href="https://login.tailscale.com/admin/machines/" target="_blank">{{ user.DisplayName }}</a>
                        <br />
                        <a @click="onLogout" style="color:green">Log out and unbind</a>
                    </div>
                </div>
            </div>
        </div>
        <!--  -->
        <div class="cbi-section">
            <h3>Global settings</h3>
            <div class="cbi-section-node">
                <!--  -->
                <div class="cbi-value">
                    <label class="cbi-value-title">Accept routes</label>
                    <div class="cbi-value-field">
                        <div class="cbi-checkbox">
                            <input name="acceptroutes" type="checkbox" :value="false" v-model="config.acceptRoutes">
                            <label></label>
                        </div>
                    </div>
                </div>
                <!--  -->
                <div class="cbi-value">
                    <label class="cbi-value-title">Device name</label>
                    <div class="cbi-value-field">
                        <div>
                            <input type="text" class="cbi-input-text" name="hostname" v-model.trim="config.hostname"
                                placeholder="Expample: OpenWrt">
                        </div>
                        <div class="cbi-value-description">
                            Empty to use the hostname
                        </div>
                    </div>
                </div>
                <!--  -->
                <div class="cbi-value">
                    <label class="cbi-value-title">Public network segment</label>
                    <div class="cbi-value-field">
                        <div>
                            <input type="text" class="cbi-input-text" name="advertiseroutes" placeholder="IP addresses,use \',\' to separate"
                                v-model.trim="config.advertiseRoutes">
                        </div>
                        <div class="cbi-value-description">
                            Network segments that will be allowed to be accessed <code>192.168.100.0/24</code>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <!--  -->
        <div class="cbi-section">
            <h3>Custom server</h3>
            <div class="cbi-section-descr">
                Use
                <a href="https://github.com/juanfont/headscale" target="_blank">headscale</a>
                to deploy your private server
            </div>
            <div class="cbi-section-node">
                <!--  -->
                <div class="cbi-value">
                    <label class="cbi-value-title">Server address</label>
                    <div class="cbi-value-field">
                        <div>
                            <input type="text" class="cbi-input-text" name="loginserver" v-model.trim="config.loginServer"
                                placeholder="Empty for not used">
                        </div>
                    </div>
                </div>
                <!--  -->
                <div class="cbi-value">
                    <label class="cbi-value-title">Token</label>
                    <div class="cbi-value-field">
                        <div>
                            <input type="password" class="cbi-input-password" name="authkey" v-model.trim="config.authkey"
                                placeholder="Empty for not used">
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <!--  -->
        <span class="cbi-page-actions control-group">
            <button class="btn cbi-button cbi-button-apply" @click="onSubmit" :disabled="disabled">SAVE & APPLY</button>
        </span>
    </div>
</template>
<script setup lang="ts">
import { ref } from 'vue';
const BASEURL = "/cgi-bin/luci/admin/services/tailscaler"
const loading = ref(true)
const disabled = ref(false)
const config = ref<ResponseConfig>({})
const status = ref<ResponseStatus>({})
const user = ref<ResponseStatusUser>({})
const userID = ref<number>()
const request = (input: string, init?: RequestInit | undefined) => {
    const uri = `${BASEURL}${input}`
    return fetch(uri, init)
}
const getStatus = async () => {
    try {
        const resp = await request("/status", {
            method: "GET"
        })
        const res = await resp.json() as ResponseStatus
        if (res) {
            status.value = res
            if (res.Self) {
                userID.value = res.Self.UserID
            }
            if (res?.User) {
                if (userID.value) {
                    const userid = userID.value
                    for (const key in res.User) {
                        if (Object.prototype.hasOwnProperty.call(res.User, key)) {
                            const element = res.User[key];
                            if (element.ID === userid) {
                                user.value = element
                                break
                            }
                        }
                    }
                }
                if (!user.value?.ID) {
                    for (const key in res.User) {
                        if (Object.prototype.hasOwnProperty.call(res.User, key)) {
                            const element = res.User[key];
                            if (element) {
                                user.value = element
                                break
                            }
                        }
                    }
                }
            } else {
                user.value = {}
            }
        } else {
            status.value = {}
        }
    } catch (error) {
        // 拿不到数据则认为程序没启动
        console.log(error);
        status.value = {}
        user.value = {}
    }
}
const getConfig = async () => {
    try {
        const resp = await request("/config", {
            method: "GET"
        })
        const res = await resp.json() as ResponseConfig
        if (res) {
            config.value = res
        }
    } catch (error) {
        console.log(error);
    }
}
const getData = async () => {
    try {
        await Promise.all([
            getConfig(),
            getStatus(),
        ])
    } catch (error) {
    } finally {
        loading.value = false
    }
}
const getInterval = async () => {
    setInterval(() => {
        getStatus()
    }, 5000)
}
getInterval()
getData()
const onSubmit = async () => {
    try {
        const resp = await request("/config", {
            method: "POST",
            headers: {
                'Content-Type': 'application/json;charset=utf-8'
            },
            body: JSON.stringify(config.value),
        })
        if (resp) {

        }
    } catch (error) {
        console.log(error);
    } finally {
        location.reload()
    }

}
const onLogout = async () => {
    if (!confirm("Do you want to log out and unbind this device?")) {
        return
    }
    try {
        const resp = await request("/logout", {
            method: "POST",
            headers: {
                'Content-Type': 'application/json;charset=utf-8'
            },
        })
        if (resp) {
        }
    } catch (error) {
        console.log(error);
    } finally {
        location.reload()
    }
}
</script>
<style lang="scss" scoped></style>