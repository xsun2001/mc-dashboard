<template>
  <div>
    <el-container>
      <el-header>
        <p>Minecraft 服务器状态</p>
      </el-header>
      <el-main style="display: flex; justify-content: center;">
        <el-space
            wrap
            fill
            size="large"
            direction="vertical"
            style="flex-grow: 0.5">
          <div v-for="s in status">
            <el-descriptions
                :column="2"
                :title="s.address + ':' + s.port"
                border>
              <div v-if="s.error">
                <el-descriptions-item label="状态">
                  <div style="color: var(--el-color-error)">获取失败</div>
                </el-descriptions-item>
              </div>
              <div v-else-if="Object.keys(s).length === 0">
                <el-descriptions-item label="状态">
                  <div style="color: var(--el-color-warning)">正在获取</div>
                </el-descriptions-item>
              </div>
              <div v-else>
                <div v-if="s.query.online">
                  <el-descriptions-item label="状态">
                    <div style="color: var(--el-color-success)">在线
                    </div>
                  </el-descriptions-item>
                  <el-descriptions-item label="游戏版本">{{ s.query.version }}</el-descriptions-item>
                  <el-descriptions-item label="在线人数">{{ s.query.players.now }}</el-descriptions-item>
                  <el-descriptions-item label="最大人数">{{ s.query.players.max }}</el-descriptions-item>
                  <el-descriptions-item label="在线玩家">{{ s.query.players.list.join(", ") }}</el-descriptions-item>
                </div>
                <div v-else>
                  <el-descriptions-item label="状态">
                    <div style="color: var(--el-color-info)">离线</div>
                  </el-descriptions-item>
                </div>
              </div>
            </el-descriptions>
          </div>
        </el-space>
      </el-main>
    </el-container>
  </div>
</template>

<script setup lang="ts">
import axios from "axios";
import {onMounted, reactive} from "vue";

interface ServerStatus {
  address: string;
  port: number;
  error: boolean;
  query: any;
}

const serverList: Array<[string, number]> = [
  ["1.mcs.xsun.io", 23301],
  ["2.mcs.xsun.io", 23301],
  ["3.mcs.xsun.io", 23301],
];

const status: Array<ServerStatus> = reactive(serverList.map(([addr, port]) => {
  return {address: addr, port: port, error: false, query: {}}
}));

onMounted(() => {
  status.forEach(s => {
    axios.get("https://mcapi.us/server/query", {params: {ip: s.address, port: s.port}})
        .then((res) => {
          s.query = res.data;
        })
        .catch((error) => {
          console.warn("Cannot access mcapi.us");
          console.warn(error);
          s.error = true;
        })
  })
})
</script>

<style>
.el-header {
  color: var(--el-color-primary-light-2);
  border-bottom: 2px solid var(--el-border-color-base);
  text-align: center;
  font-size: larger;
}

.el-descriptions {
  box-shadow: var(--el-box-shadow-base);
  border: 1px solid var(--el-border-color-base);
  border-radius: 10px;
  padding: 10px;
}
</style>
