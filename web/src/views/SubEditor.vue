<template>
  <v-container>
    <FloatMenu class="floatActionBtn">
      <v-btn fab small @click.stop="dialog = true">
        <v-icon color="purple lighten-1">mdi-plus-circle</v-icon>
      </v-btn>
      <v-btn fab small @click="save">
        <v-icon color="teal lighten-1">mdi-content-save</v-icon>
      </v-btn>
    </FloatMenu>

    <v-dialog v-model="dialog" scrollable>
      <v-card>
        <v-card-title>选择节点操作</v-card-title>
        <v-divider></v-divider>
        <v-card-text>
          <v-radio-group v-model="selectedProcess" dense>
            <v-radio
              v-for="(k, idx) in Object.keys(availableProcessors)"
              :key="idx"
              :label="availableProcessors[k].name"
              :value="k"
            ></v-radio>
          </v-radio-group>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" text @click="addProcess(selectedProcess)"
            >确认
          </v-btn>
          <v-btn text @click="dialog = false">取消</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model = "showShareDialog" max-width = "400px">
      <v-card class = "pl-4 pr-4 pb-4 pt-4">
        <v-card-title>
          <v-icon left>mdi-file-import</v-icon>
          配置导入
          <v-spacer />
        </v-card-title>
        <v-textarea v-model = "imported" :rules = "validations.importRules"
                    clear-icon = "clear" clearable label = "粘贴配置以导入" rows = "5"
                    solo
        />
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color = "primary" text @click = "importConf">确认</v-btn>
          <v-btn text @click = "showShareDialog = false">取消</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-card class="mb-4">
      <v-subheader>订阅配置</v-subheader>
      <v-form v-model="formState.basicValid" class="pl-4 pr-4 pb-0">
        <v-text-field
          v-model="options.name"
          class="mt-2"
          clear-icon="clear"
          clearable
          label="订阅名称"
          placeholder="填入订阅名称，名称需唯一"
          required
        />
        <v-text-field
          v-model="options['display-name']"
          class="mt-2"
          clear-icon="clear"
          clearable
          label="订阅显示名称"
          placeholder="填入订阅显示名称，名称无需唯一"
        />
        <!--For Subscription-->
        <v-radio-group
          v-if="!isCollection"
          v-model="options.source"
          class="mt-0 mb-0"
        >
          <template v-slot:label>
            <div>订阅来源</div>
          </template>
          <v-row dense>
            <v-col>
              <v-radio label="远程" value="remote" />
            </v-col>
            <v-col>
              <v-radio label="本地" value="local" />
            </v-col>
            <v-col></v-col>
          </v-row>
        </v-radio-group>
        <v-textarea
          v-if="!isCollection && options.source !== 'local'"
          v-model="options.url"
          :rules="validations.urlRules"
          auto-grow
          class="mt-0"
          clear-icon="clear"
          clearable
          label="订阅链接"
          placeholder="填入机场原始订阅链接"
          required
          rows="2"
        />
        <v-textarea
          v-if="options.source === 'local'"
          v-model="options.content"
          clear-icon="clear"
          clearable
          label="订阅内容"
          placeholder="填入原始订阅内容"
          autogrow
          rows="5"
          row-height="15"
          class="mt-0"
        >
        </v-textarea>
        <v-textarea
          v-if="!isCollection && options.source !== 'local'"
          v-model="options.ua"
          auto-grow
          class="mt-2"
          clear-icon="clear"
          clearable
          label="User-Agent"
          placeholder="自定义下载订阅使用的User-Agent，可选。"
          rows="2"
        />
        <!--For Collection-->
        <v-list v-if="isCollection" dense>
          <v-subheader class="pl-0">包含的订阅</v-subheader>
          <v-list-item v-for="sub in availableSubs" :key="sub.name">
            <v-list-item-avatar>
              <v-icon v-if="!sub.icon" color="teal darken-1">mdi-cloud</v-icon>
              <v-img v-else :class="getIconClass(sub.icon)" :src="sub.icon" />
            </v-list-item-avatar>
            <v-list-item-content>
              <template v-if="sub['display-name']">
                {{ sub['display-name'] }} ({{ sub.name }})
              </template>
              <template v-else>
                {{ sub.name }}
              </template>
            </v-list-item-content>
            <v-spacer></v-spacer>
            <v-checkbox v-model="selected" :value="sub.name" class="pr-1" />
          </v-list-item>
        </v-list>
        <v-textarea
          v-model="options.icon"
          auto-grow
          class="mt-2"
          clear-icon="clear"
          clearable
          label="图标链接"
          placeholder="填入想要展示的图标链接，可选。"
          rows="2"
        />
      </v-form>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn text small @click = "save" class="fixedActionBtn">
          <v-icon>mdi-content-save</v-icon>
          保存
        </v-btn>
        <v-btn text small @click.stop = "showShareDialog = true" class = "fixedActionBtn">
          <v-icon>mdi-file-import</v-icon>
          导入
        </v-btn>
        <v-btn text small @click = "share" class = "fixedActionBtn">
          <v-icon>mdi-share-circle</v-icon>
          分享
        </v-btn>

      </v-card-actions>
    </v-card>
    <v-card class="mb-4">
      <v-subheader>常用选项</v-subheader>
      <v-form class="pl-4 pr-4">
        <v-item-group>
          <v-radio-group v-model="options.useless" class="mt-0 mb-0" dense>
            过滤非法节点
            <v-row>
              <v-col>
                <v-radio label="保留" value="KEEP" />
              </v-col>
              <v-col>
                <v-radio label="删除" value="REMOVE" />
              </v-col>
              <v-col></v-col>
            </v-row>
          </v-radio-group>

          <v-radio-group v-model="options.udp" class="mt-0 mb-0" dense>
            UDP转发
            <v-row>
              <v-col>
                <v-radio label="默认" value="DEFAULT" />
              </v-col>
              <v-col>
                <v-radio label="强制开启" value="FORCE_OPEN" />
              </v-col>
              <v-col>
                <v-radio label="强制关闭" value="FORCE_CLOSE" />
              </v-col>
            </v-row>
          </v-radio-group>

          <v-radio-group
            v-model="options['skip-cert-verify']"
            class="mt-0 mb-0"
            dense
          >
            跳过证书验证
            <v-row>
              <v-col>
                <v-radio label="默认" value="DEFAULT" />
              </v-col>
              <v-col>
                <v-radio label="强制跳过" value="FORCE_OPEN" />
              </v-col>
              <v-col>
                <v-radio label="强制验证" value="FORCE_CLOSE" />
              </v-col>
            </v-row>
          </v-radio-group>
          <v-radio-group v-model="options.tfo" class="mt-0 mb-0" dense>
            TCP Fast Open
            <v-row>
              <v-col>
                <v-radio label="默认" value="DEFAULT" />
              </v-col>
              <v-col>
                <v-radio label="强制开启" value="FORCE_OPEN" />
              </v-col>
              <v-col>
                <v-radio label="强制关闭" value="FORCE_CLOSE" />
              </v-col>
            </v-row>
          </v-radio-group>
          <v-radio-group v-model="options['aead']" class="mt-0 mb-0" dense>
            Vmess AEAD
            <v-row>
              <v-col>
                <v-radio label="默认" value="DEFAULT" />
              </v-col>
              <v-col>
                <v-radio label="强制开启" value="FORCE_OPEN" />
              </v-col>
              <v-col>
                <v-radio label="强制关闭" value="FORCE_CLOSE" />
              </v-col>
            </v-row>
          </v-radio-group>
        </v-item-group>
      </v-form>
    </v-card>

    <v-card class="mb-4">
      <v-subheader>Surge 选项</v-subheader>
      <v-form class="pl-4 pr-4">
        <v-radio-group
          v-model="options['surge-hybrid']"
          class="mt-0 mb-0"
          dense
        >
          Hybrid 策略
          <v-row>
            <v-col>
              <v-radio label="默认" value="DEFAULT" />
            </v-col>
            <v-col>
              <v-radio label="强制开启" value="FORCE_OPEN" />
            </v-col>
            <v-col>
              <v-radio label="强制关闭" value="FORCE_CLOSE" />
            </v-col>
          </v-row>
        </v-radio-group>
      </v-form>
    </v-card>
    <v-card id="processors" class="mb-4">
      <v-subheader> 节点操作
        <v-spacer></v-spacer>
      <v-btn icon color = "primary" @click.stop = "dialog = true"
      >
        <v-icon>mdi-plus-circle</v-icon>
      </v-btn>
      </v-subheader>
      <!--<v-divider></v-divider>-->
      <component
        :is="p.component"
        v-for="p in processors"
        :key="p.id"
        :args="p.args"
        @dataChanged="dataChanged"
        @deleteProcess="deleteProcess"
        @down="moveDown"
        @up="moveUp"
      >
      </component>
    </v-card>
  </v-container>
</template>

<script>
import { showError, showInfo } from "@/utils";
import FloatMenu from "@/components/FloatMenu.vue";
import TypeFilter from "@/components/TypeFilter";
import RegionFilter from "@/components/RegionFilter";
import RegexFilter from "@/components/RegexFilter";
import SortOperator from "@/components/SortOperator";
import RegexRenameOperator from "@/components/RegexRenameOperator";
import RegexDeleteOperator from "@/components/RegexDeleteOperator";
import FlagOperator from "@/components/FlagOperator";
import ScriptFilter from "@/components/ScriptFilter";
import ScriptOperator from "@/components/ScriptOperator";
import RegexSortOperator from "@/components/RegexSortOperator";
import HandleDuplicateOperator from "@/components/HandleDuplicateOperator";
import ResolveDomainOperator from "@/components/ResolveDomainOperator";

const AVAILABLE_PROCESSORS = {
  "Flag Operator": {
    component: "FlagOperator",
    name: "国旗",
  },
  "Type Filter": {
    component: "TypeFilter",
    name: "类型过滤器",
  },
  "Region Filter": {
    component: "RegionFilter",
    name: "区域过滤器",
  },
  "Regex Filter": {
    component: "RegexFilter",
    name: "正则过滤器",
  },
  "Sort Operator": {
    component: "SortOperator",
    name: "节点排序",
  },
  "Regex Sort Operator": {
    component: "RegexSortOperator",
    name: "正则排序",
  },
  "Regex Rename Operator": {
    component: "RegexRenameOperator",
    name: "正则重命名",
  },
  "Regex Delete Operator": {
    component: "RegexDeleteOperator",
    name: "删除正则",
  },
  "Handle Duplicate Operator": {
    component: "HandleDuplicateOperator",
    name: "节点去重",
  },
  "Resolve Domain Operator": {
    component: "ResolveDomainOperator",
    name: "节点域名解析",
  },
  "Script Filter": {
    component: "ScriptFilter",
    name: "脚本过滤器",
  },
  "Script Operator": {
    component: "ScriptOperator",
    name: "脚本操作",
  },
};

export default {
  props: {

    isCollection: {
      type: Boolean,
      default() {
        return false;
      },
    },
  },
  components: {
    FlagOperator,
    RegexFilter,
    RegionFilter,
    TypeFilter,
    SortOperator,
    RegexSortOperator,
    RegexRenameOperator,
    RegexDeleteOperator,
    ScriptFilter,
    ScriptOperator,
    HandleDuplicateOperator,
    ResolveDomainOperator,
    FloatMenu,
  },
  data: function () {
    return {
      selectedProcess: null,
      showShareDialog: false,
      imported: "",
      dialog: false,
      validations: {
        urlRules: [
          (v) =>
            this.options.source === "remote" && (!!v || "订阅链接不能为空！"),
          (v) =>
            this.options.source === "remote" &&
            (/^https?:\/\//.test(v) || "订阅链接不合法！"),
        ],
        importRules: [(v) => !!v || "不能导入空配置！"],
      },
      formState: {
        basicValid: false,
      },
      options: {
        name: "",
        "display-name": "",
        source: "",
        url: "",
        content: "",
        icon: "",
        ua: "",
        useless: "KEEP",
        udp: "DEFAULT",
        "skip-cert-verify": "DEFAULT",
        tfo: "DEFAULT",
        "surge-hybrid": "DEFAULT",
        aead: "DEFAULT",
      },
      process: [],
      selected: [],
    };
  },

  created() {
    const name = decodeURIComponent(this.$route.params.name);
    let source;
    if (this.isCollection) {
      source =
        typeof name === "undefined" || name === "UNTITLED"
          ? {}
          : this.$store.state.collections[name];
      this.$store.commit(
        "SET_NAV_TITLE",
        source.name ? `组合订阅编辑 ➤ ${source.name}` : "新建组合订阅"
      );
      this.selected = source.subscriptions || [];
    } else {
      source =
        typeof name === "undefined" || name === "UNTITLED"
          ? {}
          : this.$store.state.subscriptions[name];
      this.$store.commit(
        "SET_NAV_TITLE",
        source.name ? `订阅编辑 ➤ ${source.name}` : "新建订阅"
      );
    }
    this.name = source.name;
    const { options, process } = loadProcess(this.options, source);
    this.options = options;
    this.process = process;
  },

  computed: {
    availableSubs() {
      return this.$store.state.subscriptions;
    },

    availableProcessors() {
      return AVAILABLE_PROCESSORS;
    },

    processors() {
      return this.process
        .filter((p) => AVAILABLE_PROCESSORS[p.type])
        .map((p) => {
          return {
            component: AVAILABLE_PROCESSORS[p.type].component,
            args: p.args,
            id: p.id,
          };
        });
    },

    config() {
      const output = {
        name: this.options.name,
        "display-name": this.options["display-name"],
        icon: this.options.icon,
        process: [],
      };
      if (this.isCollection) {
        output.subscriptions = this.selected;
      } else {
        output.url = this.options.url;
        output.source = this.options.source;
        output.content = this.options.content;
      }
      // assign user-agent, if ua is set
      let ua = this.options.ua;
      if (typeof ua != "undefined" && ua != null && ua.trim().length > 0) {
        output.ua = ua;
      } else {
        output.ua = "";
      }
      // useless filter
      if (this.options.useless === "REMOVE") {
        output.process.push({
          type: "Useless Filter",
        });
      }
      // udp, tfo, scert, surge-hybrid, aead
      for (const opt of [
        "udp",
        "tfo",
        "skip-cert-verify",
        "surge-hybrid",
        "aead",
      ]) {
        if (this.options[opt] !== "DEFAULT") {
          output.process.push({
            type: "Set Property Operator",
            args: { key: opt, value: this.options[opt] === "FORCE_OPEN" },
          });
        }
      }
      for (const p of this.process) {
        output.process.push(p);
      }
      return output;
    },
  },
  methods: {
    getIconClass(url) {
      return url.indexOf("#invert") !== -1 && !this.$vuetify.theme.dark
        ? "invert"
        : "";
    },
    save() {
      if (this.isCollection) {
        if (this.options.name && this.selected) {
          if (this.$route.params.name === "UNTITLED") {
            this.$store
              .dispatch("NEW_COLLECTION", this.config)
              .then(() => {
                showInfo(`成功创建组合订阅：${this.name}！`);
                this.$router.back();
              })
              .catch(() => {
                showError(`发生错误，无法创建组合订阅！`);
              });
          } else {
            this.$store
              .dispatch("UPDATE_COLLECTION", {
                name: this.$route.params.name,
                collection: this.config,
              })
              .then(() => {
                showInfo(`成功保存组合订阅：${this.name}！`);
              })
              .catch(() => {
                showError(`发生错误，无法保存组合订阅！`);
              });
          }
        }
      } else {
        console.log("Saving subscription...");
        if (!this.options.name) {
          showError(`订阅名字不能为空！`);
          return;
        }
        if (this.options.source === "remote" && !this.options.url) {
          showError(`订阅链接不能为空！`);
          return;
        }

        if (this.$route.params.name !== "UNTITLED") {
          this.$store
            .dispatch("UPDATE_SUBSCRIPTION", {
              name: this.$route.params.name,
              sub: this.config,
            })
            .then(() => {
              showInfo(`成功保存订阅：${this.options.name}！`);
            })
            .catch(() => {
              showError(`发生错误，无法保存订阅！`);
            });
        } else {
          this.$store
            .dispatch("NEW_SUBSCRIPTION", this.config)
            .then(() => {
              showInfo(`成功创建订阅：${this.options.name}！`);
              this.$router.back();
            })
            .catch(() => {
              showError(`发生错误，无法创建订阅！`);
            });
        }
      }
    },

    share() {
      let config = this.config;
      config.name = "「订阅名称」";
      if (this.isCollection) {
        config.subscriptions = [];
      } else {
        config.url = "「订阅链接」";
      }
      config = JSON.stringify(config);
      this.$clipboard(config);
      this.$store.commit(
        "SET_SUCCESS_MESSAGE",
        "导出成功，订阅已复制到剪贴板！"
      );
      // this.showShareDialog = false;
    },

    importConf() {
      if (this.imported) {
        const sub = JSON.parse(this.imported);
        const { options, process } = loadProcess(this.options, sub);
        delete options.name;
        delete options.url;
        delete options.content;

        Object.assign(this.options, options);
        this.process = process;

        this.$store.commit("SET_SUCCESS_MESSAGE", "成功导入订阅！");
        this.showShareDialog = false;
      } else {
        this.$store.commit("SET_ERROR_MESSAGE", "不能导入空配置！");
      }
    },

    dataChanged(content) {
      let index = 0;
      for (; index < this.process.length; index++) {
        if (this.process[index].id === content.idx) {
          break;
        }
      }
      this.process[index].args = content.args;
    },

    addProcess(type) {
      this.process.push({ type, id: uuidv4() });
      this.dialog = false;
    },

    deleteProcess(id) {
      let index = 0;
      for (; index < this.process.length; index++) {
        if (this.process[index].id === id) {
          break;
        }
      }
      this.process.splice(index, 1);
    },

    moveUp(id) {
      let index = 0;
      for (; index < this.process.length; index++) {
        if (this.process[index].id === id) {
          break;
        }
      }
      if (index === 0) return;
      // otherwise swap with previous one
      const prev = this.process[index - 1];
      const cur = this.process[index];
      this.process.splice(index - 1, 2, cur, prev);
    },

    moveDown(id) {
      let index = 0;
      for (; index < this.process.length; index++) {
        if (this.process[index].id === id) {
          break;
        }
      }
      // otherwise swap with latter one
      const cur = this.process[index];
      const next = this.process[index + 1];
      this.process.splice(index, 2, next, cur);
    },
  },
};

function loadProcess(options, source, isCollection = false) {
  options = {
    ...options,
    name: source.name,
    "display-name": source["display-name"],
    icon: source.icon,
    ua: source.ua,
  };
  if (isCollection) {
    options.subscriptions = source.subscriptions;
  } else {
    options.url = source.url;
    options.source = source.source || "remote";
    options.content = source.content;
  }
  let process = [];

  // flag
  for (const p of source.process || []) {
    switch (p.type) {
      case "Useless Filter":
        options.useless = "REMOVE";
        break;
      case "Set Property Operator":
        options[p.args.key] = p.args.value ? "FORCE_OPEN" : "FORCE_CLOSE";
        break;
      default:
        p.id = uuidv4();
        process.push(p);
    }
  }
  return { options, process };
}

function uuidv4() {
  return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(/[xy]/g, function (c) {
    var r = (Math.random() * 16) | 0,
      v = c === "x" ? r : (r & 0x3) | 0x8;
    return v.toString(16);
  });
}
</script>

<style lang="scss" scoped>
.invert {
  filter: invert(100%);
}
.fixedActionBtn{
  &.theme--dark{
    color: #ffffffcc;
  }

  &.theme--light {
    color : #00000099;
  }
}
</style>
