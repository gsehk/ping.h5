<template>
	<div class="userProfile" :class="$style.userProfile">
		<div class="commonHeader">
      <Row :gutter="24">
        <Col span="5" @click.native="goTo(-1)"  style="display: flex; justify-content: flex-start">
          <BackIcon height="21" width="21" color="#999" />
        </Col>
        <Col span="14" class="title-col">
          个人资料
        </Col>
        <Col span="5" style="display: flex; justify-content: flex-end">
          <span @click="done(canSave)" class="operate" :class="{active: canSave, disabled: !canSave}">完成</span>
        </Col>
      </Row>
    </div>
    <div class="userProfile-background-color" :class="$style.userProfileAvatar">
      <Row :gutter="24" :class="$style.rowCenter" style="position: relative;">
        <div style="width: 100%; display: flex; align-items: center;">
          <!-- 头像 -->
          <Col span="5" :class="$style.colCenter">
            <img v-lazy="avatar" :class="$style.avatar" alt="name">
          </Col>
          <!--昵称+简介-->
          <Col span="14">
            更换头像
          </Col>
          <Col span="5" :class="$style.rightIcon">
            <RightArrowIcon height="18" width="18" color="#999" />
          </Col>
        </div>
        <input ref="avatarInput" style="position:absolute; width:100%; left:0; right:0; height:100%;opacity:0;" type="file" value="" name="image" accept="image/*"
          @change="setImage"
        />
      </Row>
    </div>
    <div class="userProfile-background-color" :class="$style.userProfileSetting">
      <Row :gutter="24" :class="$style.rowCenter">
        <Col span="5">
          用户名
        </Col>
        <Col span="14">
          <input style="padding: 0;" type="text" v-model.trim="name" placeholder="输入新用户名" />
        </Col>
        <Col span="5" :class="$style.rightIcon"  @click.native="clean('name')">
          <CloseIcon height="18" width="18" color="#999" />
        </Col>
      </Row>
      <Row :gutter="24" :class="$style.rowCenter" @click.native="showSexPopup" >
        <Col span="5">
          性别
        </Col>
        <Col span="14">
          <span :class="{sexDefaultText: !sex}">{{ sexText }}</span>
        </Col>
        <Col span="5" :class="$style.rightIcon">
          <RightArrowIcon height="18" width="18" color="#999" />
        </Col>
      </Row>
      <Row :gutter="24" :class="$style.rowCenter" @click.native="handleAreaSelect(true)">
        <Col span="5">
          城市
        </Col>
        <Col span="14">
          <span :class="{sexDefaultText: !location}">{{ areaText }}</span>
        </Col>
        <Col span="5" :class="$style.rightIcon">
          <RightArrowIcon height="18" width="18" color="#999" />
        </Col>
      </Row>
      <Row :gutter="24" :class="$style.rowCenter">
        <Col span="5">
          简介
        </Col>
        <Col span="19" :class="$style.colBottom">
          <Input 
            :class="$style.intro" 
            v-model="bio" 
            type="textarea" 
            :autosize="{minRows: 1, maxRows: 4}" 
            placeholder="编辑简介"
            :maxlength="50"
          >
          </Input>
        </Col>
      </Row>
    </div>
    <mt-popup
      v-model="isShowSexPopup"
      position="bottom"
      style="width: 100%;"
      :class="$style.SexPopup"
    >
      <div>
        <Button 
          @click="setSex(1)" 
          size="large" 
          :class="$style.sexOptions" 
          type="text" :long="true"
        >
          男
        </Button>
        <Button 
          @click="setSex(2)" 
          size="large" 
          :class="$style.sexOptions" 
          type="text" 
          :long="true"
        >
          女
        </Button>
        <Button 
          @click="setSex(0)" 
          size="large" 
          :class="$style.sexOptions" 
          type="text" 
          :long="true"
        >
          保密
        </Button>
        <Button 
          @click="hideSexPopup" 
          size="large" 
          :class="$style.sexOptions" 
          type="text" 
          :long="true"
        >
          取消
        </Button>
      </div>
    </mt-popup>
    <transition 
      name="custom-classes-transition" 
      enter-active-class="animated slideInUp" 
      leave-active-class="animated slideOutDown"
    >
      <div :class="$style.avatarSelect" v-show="isShowCropper">
        <div class="avatarOp commonHeader">
          <Row 
            :gutter="24"
            style="display: flex; align-items: center;"
          >
            <Col span="5" style="display: flex; justify-content: flex-start;" @click.native="handleHideAvatarSelect">
                <BackIcon height="21" width="21" color="#999" />
            </Col>
            <Col span="14" class="title-col">
              更换头像
            </Col>
            <Col span="5" style="display: flex; justify-content: flex-end">
              <LoadingWhiteIcon height="21" width="21" v-if="loading" />
              <span v-else @click="getCropData" class="operate avatarDone">完成</span>
            </Col>
          </Row>
        </div>
        <div :class="$style.cropper">
          <vue-cropper
            class="canvasAvatar"
            ref="cropper"
            :aspect-ratio="1"
            :view-mode="1"
            :auto-crop="true"
            :auto-crop-area="0.5" 
            :minContainerWidth="minContainerWidth"
            :minContainerHeight="minContainerHeight"
            :minCanvasWidth="minContainerWidth"
            :minCanvasHeight="minContainerHeight"
            :minCropBoxWidth="200"
            :minCropBoxHeigh="200"
            drag-mode="move"
            :background="true"
            :src="imgSrc"
            :guides="false"
          >
          </vue-cropper>
        </div>
      </div>
    </transition>
      <!-- 选择城市 -->
      <search-pop v-model="areaAbout.showAreaSelect" :position="`right`" :searchfor="`USER_CITY`">
        <template slot-scope="search">
          <SearchCity :keyword="search.keyword" :datas="search.datas" @closeSearch="areaAbout.showAreaSelect=false" v-model="areaAbout.location" />
        </template>
      </search-pop>
      <!-- /选择城市 -->
	</div>
</template>
<script>
  import buildURL from 'axios/lib/helpers/buildURL';
  import { changeUrl, goTo } from '../utils/changeUrl';
  import { createAPI, addAccessToken, createOldAPI } from '../utils/request';
  import localEvent from '../stores/localStorage';
  import { getLoggedUserInfo } from '../utils/user';
  import VueCropper from 'vue-cropperjs';
  import { NOTICE } from '../stores/types';
  import md5 from 'js-md5';
  import { createUploadTask, uploadFile, noticeTask, dataURItoBlob } from '../utils/upload';
  import { Base64 } from 'js-base64';
  import getImage from '../utils/getImage';
  import contains from '../utils/contains';
  import BackIcon from '../icons/Back';
  import CloseIcon from '../icons/Close';
  import RightArrowIcon from '../icons/RightArrow';
  import lodash from 'lodash';
  import LoadingWhiteIcon from '../icons/LoadingWhite';
  import { resolveImage } from '../utils/resource';

  import SearchPop from './SearchPop/SearchPop';
  import SearchCity from './SearchPop/SearchCity';

  import { mapState } from 'vuex';

  const defaultAvatar = resolveImage(require('../statics/images/defaultAvatarx2.png'));

  // 昵称验证规则
  const usernameReg = /^[a-zA-Z_\u4E00-\u9FA5\uF900-\uFA2D][a-zA-Z0-9_\u4E00-\u9FA5\uF900-\uFA2D]*$/;
  const UserSetting = {
    components: {
      VueCropper,
      BackIcon,
      SearchPop,
      CloseIcon,
      RightArrowIcon,
      LoadingWhiteIcon,
      SearchCity
    },
    data: () => ({
      name: '', // nickname
      sex: 0, // sex
      location: '', // 地区显示字符串
      isShowSexPopup: false, // is show sex select
      bio: '', // intro
      isShowCropper: false, // is show avatar cropper
      imgSrc: '', // source image for cropper
      loading: false, // doing status
      areaAbout: { // datas for area
        showAreaSelect: false,
        location: "",
      },
      minContainerWidth: window.innerWidth,
      minContainerHeight: window.innerHeight - 46

    }),
    methods: {
      changeUrl,
      goTo,
      handleAreaSelect (status) {
        this.areaAbout.showAreaSelect = !!status;
      },
      done (status) {
        status && this.save();
      },
      cleanSetting () {
        this.imgSrc = '';
      },
      save () {
        // 新数据
        let newName = this.name;
        let newSex = this.sex;
        let newBio = this.bio;
        let newProvince = this.areaAbout.province;
        let newCity = this.areaAbout.city;
        let newLocation = this.areaAbout.location;

        // 旧数据
        let oldName = this.userInfo.name;
        let oldProvince = this.province;
        let oldCity = this.city;
        let oldLocation = this.location;
        const { datas: { sex = 0 } = {} } = this.userInfo;
        let oldSex = sex;
        const { datas: { bio = '' } = {} } = this.userInfo;
        let oldBio =  bio;

        let saveData = {};

        if(newName != oldName) {
          saveData.name = newName;
        }
        if(newProvince != oldProvince && newProvince != 0) {
          saveData.province = newProvince;
        }

        if(newLocation != oldLocation && newLocation != '')
        {
          saveData.location = newLocation;
        }

        if(newCity != oldCity && newCity != 0) {
          saveData.city = newCity;
        }

        if(newBio != oldBio) {
          saveData.bio = newBio;
        }
        if(oldSex != newSex) {
          saveData.sex = newSex;
        }
        addAccessToken().patch(
          createAPI('user'),
          {
            ...saveData
          },
          {
            validateStatus: status => status === 204
          }
        )
        .then(response => {
          // getLoggedUserInfo().then(user => {
          //   this.userInfo = { ...this.userInfo, ...user };
          // });

          this.$store.dispatch("UPDATE_INFO_OF_MINE", {...this.userInfo, ...saveData});
          this.$Message.success('修改成功');
          this.cleanSetting();
        })
      },
      clean (param) {
        this[param] = '';
      },
      showSexPopup () {
        this.isShowSexPopup = true;
      },
      hideSexPopup () {
        this.isShowSexPopup = false;
      },
      setSex(sex) {
        this.sex = sex;
        this.hideSexPopup();
      },
      setImage (e) {
        const file = e.target.files[0];

        // 判断上传文件是否为图片
        if (file.type.indexOf('image/') === -1) {
          this.$store.dispatch(NOTICE, cb => {
            cb({
              show: true,
              status: false,
              text: '请选择图片',
              time: 1500
            })
          });
          return;
        }

        // ios 9.3... typeof FileReader => 'object'
        if (typeof FileReader === 'undefined') {
            this.$store.dispatch(NOTICE, cb => {
              cb({
                show: true,
                status: false,
                text: '系统太老了...',
                time: 1500
              })
            });
            return false;
        } else {
          const reader = new window.FileReader();
          reader.onload = (event) => {
            this.imgSrc = event.target.result;
            this.$refs.cropper.replace(event.target.result);
            this.showCropper();
          };
          reader.readAsDataURL(file);
        }
      },
      // 获取裁剪后的图片信息
      getCropData () {
        let reg = /data:(.*?);/;
        let base64Reg = /^data:(.*?);base64,/;
        let fileUpload = {};
        let fileName = this.$refs.avatarInput.value;

        // 检测选中图片的mime-type;
        let mime_type = this.$refs.cropper.$refs.img.currentSrc.match(reg)[1];
        // 获取本地文件名
        fileUpload.origin_filename = fileName.replace('C:\\fakepath\\', '');
        let fileStreamData = this.$refs.cropper.getCroppedCanvas({width: 500, height: 500}).toDataURL(mime_type);

        this.loading = true;
        const formdata = new FormData();
        formdata.append('avatar', dataURItoBlob(fileStreamData));
        addAccessToken().post(createAPI('user/avatar'),
          formdata,
          {
            validateStatus: status => status === 204
          }
        )
        .then( ({ data = {} }) => {
          this.handleHideAvatarSelect();
          this.userInfo.avatar = buildURL(createAPI(`users/${TS_WEB.currentUserId}/avatar?s=200`));
          this.$store.dispatch(NOTICE, cb => {
            cb({
              text: '头像上传成功',
              time: 2500,
              status: true
            });
          });
        })
      },
      handleHideAvatarSelect () {
        this.isShowCropper = false;
        this.imgSrc = '';
        this.$refs.avatarInput.value = '';
        this.loading = false;
      },
      showCropper () {
        this.isShowCropper = true;
      },
    },
    computed: {

      ...mapState({
        userInfo: s => s.users.mine,
      }),
      currentUser(){ return this.userInfo.id },
      avatar () {
        const { avatar = defaultAvatar } = this.userInfo;

        return avatar;
      },
      canClean () {
        return this.name.length > 0;
      },
      sexText () {
        let sexObj = {
          1: '男',
          2: '女',
          0: '保密'
        };
        return this.sex ? sexObj[this.sex] : '选择性别';
      },
      areaText () {
        let text = this.location ? this.location : '选择居住地';
        if(this.areaAbout.location) {
          text = this.areaAbout.location;
        } else {
          const { datas: { location: { value: location = '' } = {} } = {} } = this.userInfo;
          if(location) {
            text = location;
          }
        }
        this.handleAreaSelect(false);
        return text;
      },

      soltAreas () {
        let areas = this.areaAbout.formateAreas;
        let provinces = this.areaAbout.formateProvince;
        if(provinces.length == 0) {
          provinces = [{name: '', id: 0}];
        }
        let citys = [];
        if(this.areaAbout.provinceText == "") {
          citys = ['北京'];
        } else {
          citys = areas[this.areaAbout.provinceText];
        }
        return [
          {
            flex: 1,
            values: provinces,
            className: 'slot1',
            textAlign: 'center'
          }, {
            flex: 1,
            values: citys,
            className: 'slot3',
            textAlign: 'center'
          }
        ]
      },
      canSave () {
        const { sex = 0 } = this.userInfo;
        const { bio = '' } = this.userInfo;
        
        let changeName = (this.name != this.userInfo.name) && usernameReg.test(this.name) && this.name.length > 2 && this.name.length < 13;
        let changeSex = this.sex != sex && this.sex != 0;
        let changeBio = this.bio != bio && this.bio != '';
        let changeProvince = this.areaAbout.province != this.province && this.areaAbout.province != 0;
        let changeCity = this.areaAbout.city != this.city && this.areaAbout.city != 0;
        let changeLocation = this.areaAbout.location != this.location && this.areaAbout.location != '';
        return (changeName || changeSex || changeBio || changeProvince || changeCity || changeLocation);
      }
    },
    mounted () {

      this.name = this.userInfo.name;
        const { 
                sex = 0, 
                bio = '',
                location = '' 
              } = this.userInfo;
        this.sex = sex;
        this.bio = bio;
        this.location = location;
    }
  };

  export default UserSetting;
</script>
<style lang="less" module>
  .areaSelect {
    position: fixed;
    left: 0;
    right: 0;
    bottom: 0;
    top: 0;
    overflow: hidden;
    margin: 0;
    background: rgba(0, 0, 0, .3);
    display: flex;
    align-items: flex-end;
  }
  .avatar {
    border-radius: 50%;
    width: 100%;
  }
  .avatarSelect {
    position: fixed;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    overflow: hidden;
    margin: 0;
    height: 100%;
    width: 100%;
    z-index: 3;
    .cropper {
      height: calc(100% - 46px);
      width: 100%;
    }
  }
  .userProfile {
    .userProfileAvatar {
      padding: 15px 0;
      border-bottom: 1px #ededed solid;
      .rowCenter {
        min-height: 50px;
        display: flex;
        align-items: center;
        .colCenter {
          display: flex;
          align-items: center;
        }
        .rightIcon {
          display: flex;
          justify-content: flex-end;
        }
      }
    }
    .userProfileSetting {
      .rowCenter {
        min-height: 50px;
        display: flex;
        align-items: center;
        border-bottom: 1px #ededed solid;
        &:last-child {
          border: none;
        }
        .colCenter {
          display: flex;
          align-items: center;
        }
        .rightIcon {
          display: flex;
          justify-content: flex-end;
        }
      }
    }
    .SexPopup {
      width: 100%;
      background: rgba(0, 0, 0, 0);
      .sexOptions {
        border-bottom: 1px solid #ededed;
        color: #333;
        border-radius: 0;
        font-size: 16px;
        &:last-child {
          margin-top: 5px;
        }
      }
    }
    .intro {
      padding-top: 5px;
      padding-bottom: 5px;
      textarea {
        border: none;
        padding-left: 0;
        padding-right: 0;
        &:hover, &:focus {
          border: none;
          box-shadow: none;
        }
      }
    }
  }
  
</style>
<style scoped lang="less">
  .avatarOp {
    height: 46px;
    border-bottom: 1px #ddd solid;
    .ivu-row {
      width: 100%;
    }
    .ivu-row, .ivu-col {
      height: 100%;
      display: flex;
      align-items: center;
      justify-content: center;
      div {
        height: 100%;
      }
    }
  }
  .canvasAvatar {
    height: 100%;
    width: 100%;
  }
  .avatarOp {
    background: #fff;
  }
  span.operate {
    font-size: 16px;
  }
  span.operate.avatarDone{
    color: #59b6d7;
  }
  span.active {
    color: #59b6d7;
  }
  span.disabled {
    color: #999;
  }
  .userProfile-background-color {
    background-color: #fff;
  }
  .sexDefaultText {
    color: #999;
  }
</style>