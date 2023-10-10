<template>
    <div id="main">
        <!--  -->
        <h2>Tailscale</h2>
        <div class="cbi-map-descr">
            Tailscale 连接您的设备，以便轻松访问远程资源。<br>
            详情请查看 <a href="https://config.com/" target="_blank">tailscale</a>
        </div>
        <!--  -->
        <div class="cbi-section">
            <h3>服务状态</h3>
            <div class="cbi-section-node">
                <div class="cbi-value cbi-value-last">
                    <label class="cbi-value-title">服务状态</label>
                    <div class="cbi-value-field">
                        <a v-if="status === undefined">加载中...</a>
                        <a v-else-if="status" style="color:green"> 运行中</a>
                        <a v-else style="color:red">未运行</a>
                        <br />
                        <a href="/admin/services/tailscale/log" target="_blank">运行日志</a>
                    </div>
                </div>
                <div class="cbi-value cbi-value-last">
                    <label class="cbi-value-title">启用</label>
                    <div class="cbi-value-field">
                        <div class="cbi-checkbox">
                            <input name="enabled" type="checkbox" :value="false" v-model="config.enable">
                            <label></label>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <!--  -->
        <div class="cbi-section">
            <h3>全局设置</h3>
            <div class="cbi-section-node">
                <!--  -->
                <div class="cbi-value">
                    <label class="cbi-value-title">接受DNS</label>
                    <div class="cbi-value-field">
                        <div class="cbi-checkbox">
                            <input name="acceptdns" type="checkbox" :value="false" v-model="config.acceptDns">
                            <label></label>
                        </div>
                    </div>
                </div>
                <!--  -->
                <div class="cbi-value">
                    <label class="cbi-value-title">接受组网</label>
                    <div class="cbi-value-field">
                        <div class="cbi-checkbox">
                            <input name="acceptroutes" type="checkbox" :value="false" v-model="config.acceptRoutes">
                            <label></label>
                        </div>
                    </div>
                </div>
                <!--  -->
                <div class="cbi-value">
                    <label class="cbi-value-title">设备名称</label>
                    <div class="cbi-value-field">
                        <div>
                            <input type="text" class="cbi-input-text" name="hostname" v-model.trim="config.hostname">
                        </div>
                        <div class="cbi-value-description">
                            留空则使用设备的名称
                        </div>
                    </div>
                </div>
                <!--  -->
                <div class="cbi-value">
                    <label class="cbi-value-title">宣告网段</label>
                    <div class="cbi-value-field">
                        <div>
                            <input type="text" class="cbi-input-text" name="advertiseroutes" placeholder="IP地址,多个实用,分开"
                                v-model.trim="config.advertiseRoutes">
                        </div>
                        <div class="cbi-value-description">
                            允许被访问的网段 <code>192.168.100.0/24</code>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <!--  -->
        <div class="cbi-section">
            <h3>自定义服务器</h3>
            <div class="cbi-section-descr">
                使用
                <a href="https://github.com/juanfont/headscale" target="_blank">headscale</a>
                部署私有服务器
            </div>
            <div class="cbi-section-node">
                <!--  -->
                <div class="cbi-value">
                    <label class="cbi-value-title">服务器地址</label>
                    <div class="cbi-value-field">
                        <div>
                            <input type="text" class="cbi-input-text" name="loginserver" v-model.trim="config.loginServer">
                        </div>
                    </div>
                </div>
                <!--  -->
                <div class="cbi-value">
                    <label class="cbi-value-title">令牌</label>
                    <div class="cbi-value-field">
                        <div>
                            <input type="password" class="cbi-input-password" name="authkey" v-model.trim="config.authkey">
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <!--  -->
        <span class="cbi-page-actions control-group">
            <button class="btn cbi-button cbi-button-apply" @click="onSubmit">保存并应用</button>
        </span>
    </div>
</template>
<script setup lang="ts">
import { ref } from 'vue';
interface TailscaleStatus {
    running?: boolean
}
interface TailscaleConfig {
    enable?: boolean
    acceptDns?: boolean
    acceptRoutes?: boolean
    hostname?: string
    advertiseRoutes?: string
    loginServer?: string
    authkey?: string
}
const status = ref<boolean | undefined>(undefined)
const config = ref<TailscaleConfig>({})
const getStatus = async () => {
    try {
        const resp = await fetch("/admin/services/tailscale/status", {
            method: "GET"
        })
        const res = await resp.json() as TailscaleStatus
        if (res) {
            status.value = res.running
        }
    } catch (error) {
        console.log(error);
    }
}
const getConfig = async () => {
    try {
        const resp = await fetch("/admin/services/tailscale/config", {
            method: "GET"
        })
        const res = await resp.json() as TailscaleConfig
        if (res) {
            config.value = res
        }
    } catch (error) {
        console.log(error);
    }
}
const getData = async () => {
    setTimeout(() => {
        getStatus()
    }, 5000)
    try {
        await Promise.all([
            getStatus(),
            getConfig(),
        ])
    } catch (error) {

    } finally {
    }
}
getData()
const onSubmit = async () => {
    try {
        const resp = await fetch("/admin/services/tailscale/config", {
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
    }

}
</script>
<style lang="scss" scoped></style>