<template>
  <!-- REST API -->
  <div class="rest-api">
    <!-- 请求头 -->
    <div class="request-line">
      <!-- 请求方法 + 请求地址 -->
      <a-input-group class="left" compact>
        <a-select size="large" v-model="request.method">
          <a-select-option value="GET">GET</a-select-option>
          <a-select-option value="POST">POST</a-select-option>
          <a-select-option value="PUT">PUT</a-select-option>
          <a-select-option value="PATCH">PATCH</a-select-option>
          <a-select-option value="DELETE">DELETE</a-select-option>
          <a-select-option value="HEAD">HEAD</a-select-option>
          <a-select-option value="CONNECT">CONNECT</a-select-option>
        </a-select>
        <a-input size="large" v-model="request.url" />
      </a-input-group>
      <!-- 提交按钮 -->
      <a-button
        v-if="!sending"
        class="submit"
        type="primary"
        size="large"
        @click="onSend"
      >
        发送
      </a-button>
      <a-button
        v-else
        class="submit"
        type="primary"
        size="large"
        @click="onCancel"
      >
        取消
      </a-button>
    </div>
    <!-- 请求体 -->
    <div class="request-body">
      <a-tabs default-active-key="1" @change="onRequestBodyTabChange">
        <a-tab-pane key="1" tab="参数" force-render>
          <RequestParams v-model="request.queryParams"></RequestParams>
        </a-tab-pane>
        <a-tab-pane key="2" tab="请求体" force-render>
          <div class="content-type">
            <div class="top-bar">
              <div class="title">内容类型</div>
              <div class="right">
                <a-dropdown :trigger="['click']">
                  <div style="cursor: pointer">
                    {{ request.contentType }} <a-icon type="down" />
                  </div>
                  <a-menu slot="overlay" @click="onContentTypeChange">
                    <a-menu-item key="无"> 无 </a-menu-item>
                    <a-menu-item key="application/json">
                      application/json
                    </a-menu-item>
                    <a-menu-item key="application/ld+json">
                      application/ld+json
                    </a-menu-item>
                    <a-menu-item key="application/hal+json">
                      application/hal+json
                    </a-menu-item>
                    <a-menu-item key="application/vnd.api+json">
                      application/vnd.api+json
                    </a-menu-item>
                    <a-menu-item key="application/xml">
                      application/xml
                    </a-menu-item>
                    <a-menu-item key="application/x-www-form-urlencoded">
                      application/x-www-form-urlencoded
                    </a-menu-item>
                    <a-menu-item key="multipart/form-data">
                      multipart/form-data
                    </a-menu-item>
                    <a-menu-item key="text/html">text/html</a-menu-item>
                    <a-menu-item key="text/plain">text/plain</a-menu-item>
                  </a-menu>
                </a-dropdown>
              </div>
            </div>

            <a-empty
              v-if="request.contentType === '无'"
              description="该请求没有任何请求体"
            >
            </a-empty>

            <RequestBody
              v-else-if="request.contentType != 'multipart/form-data'"
              :mode="request.contentType"
              v-model="request.body"
            ></RequestBody>

            <RequestMultipartFormData v-else v-model="request.formData">
            </RequestMultipartFormData>
          </div>
        </a-tab-pane>
        <a-tab-pane key="3" tab="请求头" force-render>
          <RequestHeaders v-model="request.headers"></RequestHeaders>
        </a-tab-pane>
        <a-tab-pane key="4" tab="授权" force-render>
          <div class="content-type">
            <div class="top-bar">
              <div class="title">授权类型</div>
              <div class="right">
                <a-dropdown :trigger="['click']">
                  <div style="cursor: pointer">
                    {{ request.contentType }} <a-icon type="down" />
                  </div>
                  <a-menu slot="overlay" @click="onContentTypeChange">
                    <a-menu-item key="无">无</a-menu-item>
                    <a-menu-item key="Basic Auth">Basic Auth</a-menu-item>
                    <a-menu-item key="Authorization Token"
                      >Authorization Token</a-menu-item
                    >
                    <a-menu-item key="OAuth 2.0">OAuth 2.0</a-menu-item>
                  </a-menu>
                </a-dropdown>
              </div>
            </div>

            <!-- TODO: 在此插入授权组建 -->
          </div>
        </a-tab-pane>
      </a-tabs>
    </div>
    <!-- 请求响应 -->
    <div class="request-response">
      <a-spin tip="加载中..." v-if="response.code != -1" :spinning="sending">
        <div class="status-line" v-if="response.status">
          <div class="item">
            <span class="label">状态:</span>
            <span class="value">{{ response.status }}</span>
          </div>
          <div class="item">
            <span class="label">时间:</span>
            <span class="value">{{ response.time }}ms</span>
          </div>
          <div class="item">
            <span class="label">大小:</span>
            <span class="value">{{ response.size }} B</span>
          </div>
        </div>

        <a-tabs default-active-key="1" @change="onRequestResponseTabChange">
          <a-tab-pane key="1" tab="JSON">
            <ResponseBody
              ref="responseBody"
              :content="response.body"
            ></ResponseBody>
          </a-tab-pane>
          <a-tab-pane key="2" tab="响应头"> </a-tab-pane>
        </a-tabs>
      </a-spin>

      <a-result
        v-else
        status="warning"
        title="无法发送请求"
        :sub-title="response.message"
      >
        <template #extra>
          <a-button type="primary"> 清空</a-button>
        </template>
      </a-result>
    </div>
  </div>
</template>

<script lang="ts">
import { HttpRequest } from "~/utils/http/request";
import Vue from "vue";
import Component from "nuxt-class-component";
import { restStore } from "~/store";

import RequestBody from "~/components/http/RequestBody.vue";
import ResponseBody from "~/components/http/ResponseBody.vue";

import RequestParams from "~/components/http/RequestParams.vue";
import RequestHeaders from "~/components/http/RequestHeaders.vue";
import RequestMultipartFormData from "~/components/http/RequestMultipartFormData.vue";

@Component({
  components: {
    RequestBody,
    ResponseBody,
    RequestParams,
    RequestHeaders,
    RequestMultipartFormData,
  },
})
export default class RestApi extends Vue {
  // 请求中
  get sending() {
    return restStore.isSending;
  }

  // 请求
  request: PostChildRequest = {
    method: "GET",
    url: "http://suggest.taobao.com/sug?code=utf-8&q=手机&callback=cb",
    contentType: "无",
    headers: [{ key: "", value: "" }],
    queryParams: [{ key: "", value: "" }],
    body: null,
    formData: [],
  };

  // 响应
  response = {
    code: null,
    message: null,
    body: null,
    status: null,
    time: null,
    size: null,
  };

  // 取消回调
  cancelFn = null;

  // 请求体-内容类型切换事件
  onContentTypeChange(event) {
    this.$set(this.request, "contentType", event.key);
  }

  /**
   * 发送请求
   */
  async onSend() {
    restStore.send();
    this.$nuxt.$loading.start();
    const { method, url, queryParams, headers, contentType, body, formData } =
      this.request;

    //key Values 数组 转 object
    const keyValueList2Object = (list) => {
      const obj = {};
      for (const item of list) {
        if (item.key != "" && item.value != "") {
          obj[item.key] = item.value;
        }
      }
      return obj;
    };

    const httpRequestObj = HttpRequest.setMethod(method as any);
    httpRequestObj.setUrl(url);
    httpRequestObj.setParams(keyValueList2Object(queryParams || {}));

    if (contentType === "multipart/form-data") {
      const _formData = new FormData();
      for (const item of formData) {
        _formData.append(
          item.key,
          item.valueType == "file" ? item.file : item.value
        );
      }
      httpRequestObj.setData(_formData);
    } else {
      httpRequestObj.setData(body);
    }

    // 设置header
    const _headers = keyValueList2Object(headers || {});
    if (contentType != "无") {
      _headers["Content-Type"] = contentType;
    }
    if (Object.keys(_headers).length > 0) {
      httpRequestObj.setHeaders(_headers);
    }

    const res = await httpRequestObj.execute(process.env.NODE_ENV, (c) => {
      this.cancelFn = c;
    });
    if (res) {
      restStore.stop();
      this.$nuxt.$loading.finish();
      this.response.code = (res as any).code;
      this.response.message = (res as any).message;
      this.response.body = res.data;
      this.response.status = res.status;
      this.response.time = (res as any).time;
      this.response.size = (res as any).size;
    }
  }

  /**
   * 取消请求
   */
  async onCancel() {
    this.cancelFn();
    restStore.stop();
  }

  onRequestBodyTabChange(event) {
    console.log(event);
  }

  onRequestResponseTabChange(event) {
    console.log(event);
  }
}
</script>

<style lang="scss" scoped>
.ant-input-group {
  display: flex !important;
}

.rest-api {
  padding: 15px 13px;
  // 请求行
  .request-line {
    display: flex;
    align-items: center;
    justify-content: flex-start;
    .left {
      display: flex !important;
      margin-right: 10px;

      .ant-select {
        width: 120px;
      }
      .ant-input {
        flex: 1;
      }
    }
    .submit {
    }
  }

  // 请求体
  .request-body {
    .query-param {
      .top-bar {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 10px;
        .left {
          .title {
            color: #7d7d7d;
          }
        }
        .right {
        }
      }
      .list {
        display: flex;
        flex-direction: column;
        .row {
          margin-bottom: 6px;
        }
      }
    }
    .content-type {
      .top-bar {
        height: 32px;
        display: flex;
        justify-content: flex-start;
        align-items: center;
        .title {
          color: #7d7d7d;
          margin-right: 20px;
        }
        .right {
        }
      }
    }
  }

  // 请求响应
  .request-response {
    margin-top: 20px;
    .status-line {
      display: flex;
      justify-content: flex-start;
      align-items: center;
      .item {
        margin-right: 18px;
        .label {
        }
        .value {
          color: #1890ff;
        }
      }
    }
  }
}
</style>
