<template>
    <div class="dropdown mybbButton" v-on:click="" v-bind:style="getStyle()">
        <div class="dropdown2">
            <input type="text" v-bind:id="'faceInput' + face.id" class="nameInput" placeholder="Кто это?" required v-on:change="updateName" v-on:keyup.enter="inputSubmit()" />
            <div class="suggestionsField" v-bind:id="'sg' + face.id">
                <span class="sgButton" v-for="(suggestion, index) in suggestions" v-on:click="suggest(suggestion, index)">{{ suggestion.avatar_name }}</span>
            </div>
        </div>
    </div>
</template>

<script>
    import axios from 'axios';

    var pixelWidth = require('string-pixel-width');

    function resizable (el, factor) {
        var int = Number(factor) || 7.7;
        function resize() {
            if (el.value == '') {
                el.style.width = '100px';
            }
            else {
                el.style.width = String(pixelWidth(el.value, {size: 20, family: 'Roboto'}) + 30) + 'px';
            }
        }
        var e = 'keyup,keypress,focus,blur,change'.split(',');
        for (var i in e) el.addEventListener(e[i],resize,false);
            resize();
    }

	export default {
		name: 'bbButton',
		props: ['srcWidth', 'srcHeight', 'face', 'avatar'],
        data() {
            return {
                name: '',
                suggestions: [],
            };
        },
        watch: {},
        mounted: function () {
            resizable(document.getElementById('faceInput' + String(this.face.id)), 13);
            if (this.avatar.name != "New Avatar" && this.avatar.name != undefined && this.avatar.name != null) {
                var inp = document.getElementById('faceInput' + this.face.id);
                inp.value = this.avatar.name;
                inp.disabled = true;
                inp.style.width = String(pixelWidth(inp.value, {size: 20, family: 'Roboto'}) + 30) + 'px';
                inp.style.color = "#CCC !important";
                inp.style.backgroundColor = "rgba(0, 0, 0, 0.5) !important";
            }
            var this_ = this;
            axios.get('http://photoclo.ru:8000/api/faces/' + this.face.id + '/suggest/', { headers: {Authorization: "Token " + localStorage.token}, params: {query: ''}}).then(function (response) {
                this_.suggestions = response.data.avatar;
                this_.setHeight();
            }).catch(function (error) {
                console.log(error);
            });
        },
		methods: {
            getStyle() {
                var x1 = this.face.bounding_box[0];
                var y1 = this.face.bounding_box[1];
                var x2 = this.face.bounding_box[2];
                var y2 = this.face.bounding_box[3];
                var height = y2 - y1;
                var width = x2 - x1;
                var st = '';
                st = st + 'height: ' + String(height * 100 / this.srcHeight) + '%;';
                st = st + 'width: ' + String(width * 100 / this.srcWidth) + '%;';
                st = st + 'left: ' + String((x1 + x2) * 50 / this.srcWidth) + '%;';
                st = st + 'top: ' + String((y1 + y2) * 50 / this.srcHeight) + '%;';
                return st;
            },
            updateName(event) {
                this.name = event.target.value;
            },
            inputSubmit() {
                if (this.name == '') {
                    return;
                }
                var inp = document.getElementById('faceInput' + this.face.id);
                inp.disabled = true;
                var this_ = this;
                this.avatar.name = this.name;
                axios({ method: 'PATCH', url: 'http://photoclo.ru:8000/api/avatars/' + this_.face.avatar + '/', headers: { Authorization: "Token " + localStorage.token}, data: {'new_name': this_.name, 'face': this_.face.id }}).then(function(response) {
                }).catch(function (error) {
                    console.log(error);
                });
            },
            suggest(suggestion, index) {
                var inp = document.getElementById('faceInput' + this.face.id);
                inp.value = suggestion.avatar_name;
                this.name = suggestion.avatar_name;
                inp.style.width = String(pixelWidth(suggestion.avatar_name, {size: 20, family: 'Roboto'}) + 20) + 'px';
                this.inputSubmit();
            },
            setHeight() {
                var dropdown = document.getElementById('sg' + this.face.id);
                dropdown.style.height = String(Math.min(window.innerHeight - dropdown.getBoundingClientRect().top, this.suggestions.length * 25)) + 'px';
            },
            updateSuggestions() {

            }
        }
	}
</script>

<style scoped>

    .dropdown {
        position: relative;
        background-color: rgba(0, 0, 0, 0) !important;
        margin: 0px !important;
        padding: 0px !important;
        border: 2px solid rgba(255, 255, 0, 0.5) !important;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .dropdown2 {
        top: 100%;
        position: relative;
    }

    .mybbButton {
        z-index: 3 !important;
    }

    .mybbButton:hover {
        border: 3px solid rgba(255, 255, 0, 1) !important;
        margin-bottom: 3px !important;
    }

    .suggestionsField {
        display: none;
        flex-direction: column;
        position: relative;
        overflow: auto;
        height: 0px;
        width: auto;
        background-color: #FFF;
    }

    .sgButton {
        z-index: 7;
        text-align: left;
        height: 25px;
        border-top: 1px solid #3A78DE;
    }

    .nameInput {
        z-index: 5 !important;
        align-self: center;
        display: none;
        font-family: 'Roboto', sans-serif;
        font-size: 20px;
        height: auto;
        border: 2px solid white;
        border-radius: 2px;
        width: 100px;
        transition: width 0.05s;
        text-align: center;
        outline: none !important;
    }

    .dropdown:hover .nameInput,
    .dropdown:focus .nameInput {
        display: block;
    }

    .suggestionsField:hover {
        display: flex;
    }

    .nameInput:focus + .suggestionsField {
        display: flex;
    }
</style>