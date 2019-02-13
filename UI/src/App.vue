<template>
    <div id="app">
        <!--<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css">-->
        <link href="https://fonts.googleapis.com/css?family=Roboto" rel="stylesheet">
        <!--<span> {{localStorage.imagasNum}} </span>-->

        <b-navbar v-if="authenticated" toggleable="md" type="light" variant="light" class="navBar">

            <b-navbar-toggle target="nav_collapse"></b-navbar-toggle>

            <b-navbar-brand style="color: black; font-family: 'Roboto', sans-serif; font-size: 30px !important;" href="photoclo.ru:8000">PHOTOCLO</b-navbar-brand>

            <b-collapse is-nav id="nav_collapse">



                <!-- Right aligned nav items -->
                <b-navbar-nav class="ml-auto">
                    <!--Search     -->
                    <!--<form style="display: flex !important; flex-direction:row-reverse !important; text-align: center !important;" class="form-inline md-form form-sm mt-0">-->
                        <!--<input style="width: 80% !important; margin-left: 10px !important;" class="form-control form-control ml-3 w-100" type="text" placeholder="Поиск" aria-label="Search">-->
                        <!--<i style="margin-left: 0px !important;" class="fas fa-search" aria-hidden="true"></i>-->
                    <!--</form>-->

                    <form style="display: flex !important; flex-direction:row-reverse !important; margin-right: 20px; margin-top: 5px !important;">
                        <input style="margin-left: 10px !important;" class="form-control form-control ml-3" type="text" placeholder="Поиск" aria-label="Search" v-on:change="updateSearch" v-on:keyup.enter="submitSearch" v-on:submit="submitSearch">
                        <i style="margin-top: 10px !important; margin-right: 0px !important; margin-left: 5px !important;" class="fas fa-search" aria-hidden="true"></i>
                    </form>


                    <div v-if="onUploading" class="uploadProcessDiv">
                        <span class="uploadProcessSpan"> Идет загрузка:  </span>
                        <b-progress :value="filesUploaded" :max="filesToUpload" class="mb-3"></b-progress>
                    </div>

                    <div v-if="!onUploading" v-on:click="startUpload" class="iconDiv">
                        <div><img src="https://i.ibb.co/vJs3zrs/icon.png"  class="iconImg"/></div>
                        <span class="iconText">Загрузка</span>
                    </div>

                    <!--<span> myUploader.data().toClose </span>-->
                    <b-modal id="uploadModal" ref="uploadModal" hide-footer=true @hide="isModalShown=false" title="Загрузка фотографий">
                        <myUploader ref="myUploader" v-on:closeModal="isModalShown = false;" url="http://photoclo.ru:8000/api/photos/"
                                    @upload-image-attempt="uploadAttempt();" @upload-image-finish="uploadFinish();"
                                    @upload-image-success='updateImagesOk();' @upload-image-failure='updateImagesFail();'
                                    @set-files-num="setFilesNum"  @start-upload="showProgress"> </myUploader>
                    </b-modal>

                    <div v-if="isDiskSynchronized" v-on:click="goToYandexDisk" title="Диск уже подключен" v-b-tooltip.hover class="iconDiv">
                        <div><img src="https://i.ibb.co/0cmbT5w/ya-disk.png" class="iconImg"></div>
                        <span  class="iconText"> Яндекс.Диск </span>
                    </div>

                    <div v-if="!isDiskSynchronized" v-on:click="goToYandexDisk" title="Подключить Яндекс.диск" v-b-tooltip.hover class="iconDiv">
                        <div><img src="https://i.ibb.co/0cmbT5w/ya-disk.png" class="iconImg"></div>
                        <span class="iconText"> Яндекс.Диск </span>
                    </div>

                    <div class="iconDiv" v-on:click="logout" id="logoutDiv">
                        <div><img src="https://i.ibb.co/nLqCDNy/logout-512.png" class="iconImg"></div>
                        <span class="iconText"> Выйти</span>
                    </div>




                    <!--<div class="iconDiv" id="userDiv">-->
                    <!--<img src="https://i.ibb.co/SDTzj5R/user-icon.png" class="iconImg" id="userImg"/>-->
                    <!--<b-dropdown id="userButton" right text=""  class="mr-sm-2" no-caret variant="link">-->
                    <!--<b-dropdown-item href="#">Профиль</b-dropdown-item>-->
                    <!--<b-dropdown-item v-on:click="logout()">Выйти</b-dropdown-item>-->
                    <!--</b-dropdown>-->
                    <!--</div>-->
                </b-navbar-nav>

            </b-collapse>
        </b-navbar>
        
        <span v-if="cnt == '0' && authenticated">GG</span>
        <router-view @authenticated="setAuthenticated" ref="child" v-bind:cnt="cnt" @cntInc="cntInc()" @cntDec="cntDec()" @cntSet="cntSet"/>

    </div>
</template>

<script>
    import axios from 'axios';
    import myUploader from './components/myUploader.vue'
    var modal = document.getElementById('id01');

    window.onclick = function(event) {
        if (event.target == modal) {
            modal.style.display = "none";
        }
    }

    window.onload = function() {
        var this_ = this;
        document.getElementsByClassName('myUploader')[0].__vue__.$props.my_header = {Authorization: "Token " + localStorage.token};
    }

    export default {
        name: 'App',
        data() {
            return {
                authenticated: false,
                token: undefined,
                isModalShown: false,
                onUploading: false,
                filesUploaded: 0,
                filesToUpload: 0,
                isDiskSynchronized: false,
                // That's bullshit because of assync of JS:
                hasJustStarted: false,
                // That's bullshit because of assync of JS:
                cnt: 0,
                search: ''
            }
        },
        components: {
            myUploader
        },
        watch: {
            isModalShown(value) {
                if (value) {
                    this.$refs.uploadModal.show();
                }
                else {
                    this.$refs.uploadModal.hide();
                }
            },
            onUploading(value) {
                if (value) {
                    window.onbeforeunload = function() {
                        alert("Вы уверены что хотите выйти? Загрузка может прерваться!");
                        return "Вы уверены что хотите выйти? Загрузка может прерваться!";
                    };
                }
                else {
                    window.onbeforeunload = null;
                }
            }
        },
        mounted() {
            var this_ = this;

            if (localStorage.hasOwnProperty('cnt')) {
                this.cnt = localStorage.cnt;
            }
            else {
                this.cnt = 0;
                localStorage.cnt = 0;
            }
            axios.get('http://photoclo.ru:8000/api/tokens/status/',{ headers: {Authorization: "Token " + String(localStorage.token)}}).then(function (response) {
                if (!response.data.sync) {
                    this_.isDiskSynchronized = false;
                } else {
                    this_.isDiskSynchronized = true;
                }
            });
            if (localStorage.hasOwnProperty('token')) {
                this.token = localStorage.token;
                this.authenticated = true;
            }
            if(!this.authenticated) {
                this.$router.replace({ name: "login" });
            }
            else {
                this.$router.replace({name: "secure"});
            }
        },
        computed: {
            cnt: function () {
                return localStorage.cnt;
            }
        },
        methods: {
            updateToken () {
                document.getElementsByClassName('myUploadBox')[0].__vue__.$props.my_header = {Authorization: "Token " + localStorage.token};
            },
            setAuthenticated(status) {
                this.authenticated = status;
            },
            logout() {
                var this_ = this;
                axios.post('http://photoclo.ru:8000/api/sign_out/', {Authorization: "Token " + String(localStorage.token)}).then(function () {
                    this_.authenticated = false;
                    this_.$router.replace({ name: "login" });
                    delete localStorage.token;
                });
            },
            updateImagesOk() {
                this.filesUploaded += 1;
                this.$refs.child.updateImages();
            },
            updateImagesFail() {
                this.$refs.child.updateImages();
            },
            resetUploader() {
                this.$refs.myUploader.resetUploader();
            },
            startUpload() {
                this.isModalShown=true;
                this.updateToken();
                this.resetUploader()
            },
            showProgress() {
                this.onUploading = true;
                this.hasJustStarted = true;
            },
            uploadAttempt() {
                this.onUploading = true;
            },
            uploadFinish() {
                if (this.hasJustStarted) {
                    this.hasJustStarted = false
                } else {
                    this.onUploading = false;
                    this.filesUploaded = 0;
                    this.filesToUpload = 0;
                }
            },
            setFilesNum(value) {
                this.onUploading = true;
                this.filesToUpload = value;
            },
            goToYandexDisk() {
                if (!this.isDiskSynchronized) {
                    axios.get('http://photoclo.ru:8000/api/tokens/code/',{ headers: {Authorization: "Token " + String(this.token)}}).then(function (response) {
                        window.location.href = response.data.url;
                    });
                }
            },
            cntInc() {
                this.cnt++;
                localStorage.cnt = this.cnt;
            },
            cntDec() {
                this.cnt--;
                localStorage.cnt = this.cnt;
            },
            cntSet(value) {
                this.cnt = Number(value);
                localStorage.cnt = this.cnt;
            },
            updateSearch(event) {
                this.search = event.target.value;
            },
            submitSearch() {
                this.$refs.child.search(this.search);
            }
        },
    }
</script>

<style>
    body {
        background-color: #EEEEEE !important;
    }
    h1 {
        padding: 0;
        margin-top: 0;
    }

    .active {
        background-color: #4CAF50;
        color: white;
    }

    button:hover {
        opacity: 0.8;
    }

    .dropAreaDragging{
        background-color: #ccc;
    }

    .navBar {
        border-bottom: 2px solid #3A78DE;
        box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
    }

    
    #userImg {
        padding-bottom: 8px;
        height: 45px;
        width: auto;
    }

    .iconDiv {
        /*border: 1px solid grey;*/
        cursor: pointer;
        margin-top: 10px;
        margin-bottom: 10px;
        margin-left: 5px;
        margin-right: 30px;
        display: flex;
        flex-direction: row;
        text-align: center;
    }

    #logoutDiv {
        margin-right: 10px !important;
    }

    .iconDiv:hover {
        background-color: rgba(58, 120, 222, 0.05) !important;
    }

    .iconText {
        font-weight: 100;
        font-family: 'Roboto', sans-serif;
        font-size: 17px;
        margin-left: 10px;
        margin-right: 10px;
    }

    .iconImg {
        width: 20px;
        height: 15px;
        margin-left: 0px;
    }

    .userImg {
        width: 20px;
        height: 20px;
        margin-left: 5px;
        margin-right: 3px;
    }

    #uploadModal {
        font-family: 'Roboto', sans-serif;
    }

    .uploadProcessDiv {
        margin-right: 30px;
        display: flex;
        flex-direction: column;
        text-align: center;
    }

    .uploadProcessSpan {
        margin-top: 5px;
    }

    #dropUser:hover .dropdown-menu{
        margin-top: 0;
        display: block;
    }

    .searchForm {
    }

    .searchField {
        width: 120vw !important;
        margin-right: 0px !important;
        margin-left: 30px !important;
    }

    .searchForm:hover,
    .searchForm:focus {
        background-color: rgba(0, 0, 0, 0) !important;
    }
</style>
