<template>
  <view class="upload">
    <view
      v-for="(item, index) in uploadFiles"
      class="upload__file"
      :class="'upload__file--' + item.status"
      :style="{
        width: addUnit(width),
        height: addUnit(height),
      }"
      @tap="onPreview(index, item)"
    >
      <slot :item="item" :index="index">
        <image
          :style="{
            width: addUnit(width),
            height: addUnit(height),
          }"
          :src="item.url"
          :mode="imageMode"
        />
        <view
          class="upload__file--mask"
          v-if="['uploading', 'fail'].includes(item.status)"
        >
          <text v-if="item.status == 'uploading'">{{ item.percentage }}%</text>
          <icon
            v-if="item.status == 'fail'"
            type="warn"
            color="red"
            size="20"
          />
        </view>
      </slot>
      <icon
        class="upload__file--remove"
        type="clear"
        color="#263240"
        size="20"
        @tap.stop="handleRemove(item)"
      />
    </view>
    <view
      :style="{
        width: addUnit(width),
        height: addUnit(height),
      }"
      class="upload__btn"
      hover-class="upload__btn--hover"
      hover-stay-time="150"
      v-if="showUploadBtn"
      @tap="chooseImage"
    >
      <slot>
        <view class="icon"></view>
      </slot>
    </view>
  </view>
</template>

<script>
function noop() {}

function uuid() {
  let d = new Date().getTime();
  let uuid = "xxxxxxxxxxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(/[xy]/g, (c) => {
    let r = (d + Math.random() * 16) % 16 | 0;
    d = Math.floor(d / 16);
    return (c == "x" ? r : (r & 0x3) | 0x8).toString(16);
  });
  return uuid;
}

export default {
  name: "hjb-upload",
  props: {
    fileList: {
      type: Array,
      default: () => [],
    },
    // 后端地址
    action: {
      type: String,
      default: "",
    },
    // 是否开启图片多选
    multiple: {
      type: Boolean,
      default: false,
    },
    sourceType: {
      type: Array,
      default: () => ["album", "camera"],
    },
    // 所选的图片的尺寸, 可选值为original compressed
    sizeType: {
      type: Array,
      default: () => ["original", "compressed"],
    },
    // 预览上传的图片时的裁剪模式，和image组件mode属性一致
    imageMode: {
      type: String,
      default: "aspectFill",
    },
    // 头部信息
    headers: {
      type: Object,
      default: () => ({}),
    },
    // 额外携带的参数
    formData: {
      type: Object,
      default: () => ({}),
    },
    // 上传的文件字段名
    name: {
      type: String,
      default: "file",
    },
    // 文件大小限制，单位为byte
    maxSize: {
      type: [String, Number],
      default: Number.MAX_VALUE,
    },
    // 最大上传数量 多图的时候才有效
    limit: {
      type: [String, Number],
      default: 10,
    },
    // 是否自动上传
    autoUpload: {
      type: Boolean,
      default: true,
    },
    // 是否显示toast消息提示
    showTips: {
      type: Boolean,
      default: true,
    },
    // 内部预览图片区域和选择图片按钮的区域宽度
    width: {
      type: [String, Number],
      default: 120,
    },
    // 内部预览图片区域和选择图片按钮的区域高度
    height: {
      type: [String, Number],
      default: 120,
    },

    // 允许上传的图片后缀
    fileType: {
      type: Array,
      default() {
        // 支付宝小程序真机选择图片的后缀为"image"
        // https://opendocs.alipay.com/mini/api/media-image
        return [];
      },
    },
    // 是否启用
    disabled: {
      type: Boolean,
      default: false,
    },
    // 是否启用预览
    enablePreview: {
      type: Boolean,
      default: true,
    },
    beforeUpload: Function,
    beforeRemove: Function,
    onRemove: {
      type: Function,
      default: noop,
    },
    onChange: {
      type: Function,
      default: noop,
    },
    onSuccess: {
      type: Function,
      default: noop,
    },
    onProgress: {
      type: Function,
      default: noop,
    },
    onError: {
      type: Function,
      default: noop,
    },
    httpRequest: {
      type: Function,
      default: uni.uploadFile,
    },
  },
  data() {
    return {
      uploadFiles: [],
      uploading: false,
      reqs: {},
    };
  },
  computed: {
    showUploadBtn() {
      if (this.multiple) {
        return this.limit > this.uploadFiles.length;
      }
      return this.uploadFiles.length < 1;
    },
  },
  watch: {
    fileList: {
      immediate: true,
      handler(files) {
        this.uploadFiles = files.map((item) => {
          item.uid = item.uid || uuid();
          item.status = item.status || "success";
          return item;
        });
      },
    },
  },
  methods: {
    // 清除列表
    clear() {
      this.uploadFiles = [];
    },
    // 该方法供用户通过ref调用，手动上传
    upload() {
      if (this.disabled) return;
      this.uploadFiles
        .filter((file) => file.status === "ready")
        .forEach((item) => {
          this.uploadFile(item);
        });
    },
    // 图片预览
    onPreview(index, file) {
      if (!this.enablePreview) return;
      const images = this.uploadFiles.map((item) => item.url || item.path);
      uni.previewImage({
        current: index,
        urls: images,
        success: () => {
          this.$emit("preview", file, index);
        },
        fail: () => {
          uni.showToast({
            title: "Preview image failed",
            icon: "none",
          });
        },
      });
    },
    // 选择图片
    chooseImage() {
      if (this.disabled) return;
      const {
        limit,
        multiple,
        maxSize,
        uploadFiles,
        sourceType,
        sizeType,
        autoUpload,
      } = this;
      const maxCount = limit - uploadFiles.length;
      uni.chooseImage({
        count: multiple ? maxCount : 1,
        sourceType,
        sizeType,
        type: "image",
        success: (res) => {
          const files = res.tempFiles;
          if (limit && uploadFiles.length + files.length > limit) {
            this.showToast("The maximum number of files allowed is exceeded");
            return;
          }
          files.forEach((val, index) => {
            // 检查文件后缀是否允许，如果不在this.fileType内，就会返回false
            if (!this.checkFileExt(val)) return;
            // 如果是非多选，index大于等于1或者超出最大限制数量时，不处理
            if (!multiple && index >= 1) return;
            if (val.size > maxSize) {
              this.showToast("Exceeds allowable file size");
              return;
            }

            val.uid = uuid();
            let file = {
              status: "ready",
              size: val.size,
              percentage: 0,
              uid: val.uid,
              raw: val,
              url: val.path,
            };
            this.uploadFiles.push(file);
            this.onChange(file, this.uploadFiles);
            if (autoUpload) this.uploadFile(file);
          });
          // 每次图片选择完，抛出一个事件，并将当前内部选择的图片数组抛出去
          this.$emit("choose-complete", this.uploadFiles);
        },
        fail: (e) => {
          this.$emit("choose-fail", e);
        },
      });
    },
    getFile(rawFile) {
      let fileList = this.uploadFiles;
      let target;
      fileList.every((item) => {
        target = rawFile.uid === item.uid ? item : null;
        return !target;
      });
      return target;
    },
    // 对失败的图片重新上传
    retry() {
      uni.showLoading({
        title: "重新上传",
      });
      this.uploadFiles
        .filter((item) => item.ststus == "fail")
        .forEach((item) => {
          this.uploadFile(item);
        });
    },
    // 上传图片
    uploadFile(rawFile) {
      // 检查上传地址
      if (!this.action) {
        this.showToast("Please configure the upload address", true);
        return;
      }
      if (!this.beforeUpload) {
        this.post(rawFile);
        return;
      }
      const before = this.beforeUpload(rawFile);
      if (before && before.then) {
        console.log("async before");
        before.then(
          (file) => {
            this.post(file);
          },
          () => {
            this.handleRemove(null, rawFile);
          }
        );
      } else if (before !== false) {
        this.post(rawFile);
      } else {
        this.handleRemove(null, rawFile);
      }
    },

    post(rawFile) {
      const { uid } = rawFile;
      const options = {
        header: this.headers,
        filePath: rawFile.url,
        formData: this.formData,
        name: this.name,
        url: this.action,
        success: (res) => {
          // 判断是否json字符串，将其转为json格式
          let data = this.jsonString(res.data)
            ? JSON.parse(res.data)
            : res.data;
          if (![200, 201, 204].includes(res.statusCode)) {
            this.uploadError(data, rawFile);
          } else {
            this.uploadSuccess(data, rawFile);
          }
          delete this.reqs[uid];
        },
        fail: (err) => {
          this.uploadError(err, rawFile);
          delete this.reqs[uid];
        },
      };
      var uploadTask = this.httpRequest(options);
      this.reqs[uid] = uploadTask;
      uploadTask.onProgressUpdate((res) => {
        this.uploadProgress(res, rawFile);
      });
    },
    // 上传中
    uploadProgress(ev, rawFile) {
      const file = this.getFile(rawFile);
      this.onProgress(ev, file, this.uploadFiles);
      file.status = "uploading";
      file.percentage = ev.progress || 0;
    },
    // 上传成功
    uploadSuccess(res, rawFile) {
      const file = this.getFile(rawFile);
      const fileList = this.uploadFiles;
      file.status = "success";
      file.response = res;
      file.name = res.name;
      file.url = res.url;
      this.onSuccess(res, file, this.uploadFiles);
      this.onChange(file, this.uploadFiles);
    },
    // 上传失败
    uploadError(err, rawFile) {
      const file = this.getFile(rawFile);
      const fileList = this.uploadFiles;
      file.status = "fail";
      fileList.splice(fileList.indexOf(file), 1);
      this.onError(err, file, this.uploadFiles);
      this.onChange(file, this.uploadFiles);
    },
    // 删除
    handleRemove(file, raw) {
      if (raw) {
        file = this.getFile(raw);
      }
      let doRemove = () => {
        this.abort(file);
        let fileList = this.uploadFiles;
        fileList.splice(fileList.indexOf(file), 1);
        this.onRemove(file, fileList);
      };

      if (!this.beforeRemove) {
        doRemove();
      } else if (typeof this.beforeRemove === "function") {
        const before = this.beforeRemove(file, this.uploadFiles);
        if (before && before.then) {
          before.then(() => {
            doRemove();
          }, noop);
        } else if (before !== false) {
          doRemove();
        }
      }
    },
    // 中断上传任务
    abort(file) {
      const { reqs } = this;
      if (file) {
        let uid = file;
        if (file.uid) uid = file.uid;
        if (reqs[uid]) {
          reqs[uid].abort();
        }
      } else {
        Object.keys(reqs).forEach((uid) => {
          if (reqs[uid]) reqs[uid].abort();
          delete reqs[uid];
        });
      }
    },
    // 判断文件后缀是否允许
    checkFileExt(file) {
      if (this.fileType.length == 0) {
        return true;
      }
      // 检查是否在允许的后缀中
      let noArrowExt = false;
      // 获取后缀名
      let fileExt = "";
      const reg = /.+\./;
      // 如果是H5，需要从name中判断
      // #ifdef H5
      fileExt = file.name.replace(reg, "").toLowerCase();
      // #endif
      // 非H5，需要从path中读取后缀
      // #ifndef H5
      fileExt = file.path.replace(reg, "").toLowerCase();
      // #endif
      // 使用数组的some方法，只要符合limitType中的一个，就返回true
      noArrowExt = this.fileType.some((ext) => {
        // 转为小写
        return ext.toLowerCase() === fileExt;
      });
      if (!noArrowExt)
        this.showToast(
          `You are not allowed to select files in ${fileExt} format`
        );
      return noArrowExt;
    },
    // 提示用户消息
    showToast(message, force = false) {
      if (this.showTips || force) {
        uni.showToast({
          title: message,
          icon: "none",
        });
      }
    },
    // 添加单位，如果有rpx，%，px等单位结尾或者值为auto，直接返回，否则加上rpx单位结尾
    addUnit(value = "auto", unit = "rpx") {
      let reg = /^(?:-?\d+|-?\d{1,3}(?:,\d{3})+)?(?:\.\d+)?$/;
      value = String(value);
      // 用uView内置验证规则中的number判断是否为数值
      return reg.test(value) ? `${value}${unit}` : value;
    },
    /**
     * 是否json字符串
     */
    jsonString(value) {
      if (typeof value == "string") {
        try {
          var obj = JSON.parse(value);
          if (typeof obj == "object" && obj) {
            return true;
          } else {
            return false;
          }
        } catch (e) {
          return false;
        }
      }
      return false;
    },
  },
};
</script>

<style lang="scss" scoped>
.upload {
  display: flex;
  flex-wrap: wrap;
  align-items: center;

  &__file,
  &__btn {
    border-radius: 4rpx;
    margin: 0 30rpx 30rpx 0;
  }

  &__file {
    position: relative;
    image {
      border-radius: 4rpx;
    }
    &--mask {
      position: absolute;
      top: 0;
      bottom: 0;
      left: 0;
      right: 0;
      background-color: rgba(37, 38, 45, 0.4);
      border-radius: 4rpx;
      display: flex;
      align-items: center;
      justify-content: center;
      color: #fff;
    }

    &--remove {
      position: absolute;
      right: 0;
      top: 0;
      transform: translate(50%, -50%);
      border: 3rpx solid #fff;
      border-radius: 50%;
      background: #fff;
    }
  }

  &__btn {
    display: flex;
    align-items: center;
    justify-content: center;
    border: 2rpx solid #e0e0e0;

    &--hover {
      background: rgba(0, 0, 0, 0.04);
    }

    .icon {
      position: relative;

      &::before,
      &::after {
        content: "";
        position: absolute;
        top: 50%;
        left: 50%;
        width: 32rpx;
        height: 3rpx;
        transform: translate(-50%, -50%);
        background-color: #b0b0b0;
      }

      &::after {
        transform: translate(-50%, -50%) rotate(90deg);
      }
    }
  }
}
</style>
