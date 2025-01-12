---
title: "EasyTier 命令生成器"
date: 2025-01-12T20:01:11+08:00
draft: false
---

简单写了一个 EasyTier 的命令和 systemd 配置的生成器，总之就是不用翻文档了

[easyTier 命令生成器](https://etc.liaowen.us.kg)

已下是源码
```html
<script setup>
import { NButton, NFlex, NButtonGroup, NConfigProvider, NGlobalStyle, NAutoComplete, NInput, NSelect, NCode, NH1 } from 'naive-ui'
import { darkTheme, lightTheme } from 'naive-ui'
import { usePreferredColorScheme, useStorage, useClipboard } from '@vueuse/core'
import { Sun, Moon, Computer } from 'lucide-vue-next'
import { computed } from 'vue'
import hljs from 'highlight.js/lib/core'
import bash from 'highlight.js/lib/languages/bash'
import ini from 'highlight.js/lib/languages/ini'
hljs.registerLanguage('bash', bash)
hljs.registerLanguage('ini', ini)

const preferredTheme = usePreferredColorScheme()
const colorMode = useStorage('color-mode', 'auto')
const ipAddress = useStorage('ip-address', '')
const peerAddress = useStorage('peer-address', '')
const isDHCP = useStorage('is-dhcp', false)
const networkName = useStorage('network-name', '')
const networkSecret = useStorage('network-secret', '')
const options = computed(() => {
    if (!peerAddress.value) return []
    const parts = peerAddress.value.split("://")
    const tmpValue = parts.length > 1 ? parts[1].split(":")[0] : parts[0].split(":")[0]
    return ["udp://" + tmpValue + ":11010", "tcp://" + tmpValue + ":11010", "ws://" + tmpValue + ":11011", "wss://" + tmpValue + ":11012"]
})
const isDHCPOptions = [
    { label: 'DHCP', value: true },
    { label: '手动', value: false },
]
const username = useStorage('username', '')

// 计算当前应该使用的主题
const currentTheme = computed(() => {
    if (colorMode.value === 'auto') {
        return preferredTheme.value === 'dark' ? darkTheme : lightTheme
    }
    return colorMode.value === 'dark' ? darkTheme : lightTheme
})
const easytierCommand = computed(() => {
    // 如果必要字段为空，返回空字符串
    if ((!isDHCP.value && !ipAddress.value) || !peerAddress.value || !username.value) {
        return ''
    }

    let command = ''

    if (username.value === 'root') {
        command = 'sudo /root/.cargo/bin/easytier-core'
    } else {
        command = 'sudo /home/' + username.value + '/.cargo/bin/easytier-core'
    }


    // 添加 IP 地址参数或 DHCP 选项
    if (isDHCP.value) {
        command += ' -d'
    } else if (ipAddress.value) {
        command += ` -i ${ipAddress.value}`
    }
    if (networkName.value && networkSecret.value) {
        // 添加网络名称和密码
        command += ` --network-name ${networkName.value}`
        command += ` --network-secret ${networkSecret.value}`
    }

    // 添加 peer 地址
    command += ` -p ${peerAddress.value}`

    return command
})

const serviceCommand = computed(() => {
    if (!easytierCommand.value) return ''
    // 移除 sudo 前缀
    return easytierCommand.value.replace('sudo ', '')
})

const easytierConfig = computed(() => {
    if (!serviceCommand.value) return ''

    return `[Unit]
Description=EasyTier Service
After=network.target syslog.target
Wants=network.target

[Service]
Type=simple
ExecStart=${serviceCommand.value}

[Install]
WantedBy=multi-user.target`
})
const copyCommand = () => {
    const { copy } = useClipboard()
    copy(easytierCommand.value)
}
const copyConfig = () => {
    const { copy } = useClipboard()
    copy(easytierConfig.value)
}
</script>

<template>
    <NConfigProvider :theme="currentTheme" :hljs="hljs">
        <NGlobalStyle />
            <NFlex vertical align="center" justify="center" size="large"
                style="height: 100vh; width: 30vw; margin: 0 auto;">
                <NH1>EasyTier 命令生成器</NH1>
                <NButtonGroup>
                    <NButton secondary @click="colorMode = 'light'"
                        :type="colorMode === 'light' ? 'primary' : 'default'">
                        <Sun />
                    </NButton>
                    <NButton secondary @click="colorMode = 'dark'" :type="colorMode === 'dark' ? 'primary' : 'default'">
                        <Moon />
                    </NButton>
                    <NButton secondary @click="colorMode = 'auto'" :type="colorMode === 'auto' ? 'primary' : 'default'">
                        <Computer />
                    </NButton>
                </NButtonGroup>
                <NSelect v-model:value="isDHCP" :options="isDHCPOptions" />
                <NInput placeholder="请输入用户名" v-model:value="username" />
                <NInput placeholder="请输入ip地址" v-model:value="ipAddress" :disabled="isDHCP" />
                <NAutoComplete placeholder="请输入peer地址" v-model:value="peerAddress" :options="options"
                    style="width: 100%;" />
                <NInput placeholder="请输入网络名" v-model:value="networkName" />
                <NInput type="password" placeholder="请输入网络密码" v-model:value="networkSecret" />
                <NCode :code="easytierCommand" language="bash" />
                <NButton @click="copyCommand">复制命令</NButton>
                <NCode :code="easytierConfig" language="ini" />
                <NButton @click="copyConfig">复制配置</NButton>
            </NFlex>
    </NConfigProvider>
</template>
```