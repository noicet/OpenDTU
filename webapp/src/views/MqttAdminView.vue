<template>
    <BasePage :title="'MqTT Settings'" :isLoading="dataLoading">
        <BootstrapAlert v-model="showAlert" dismissible :variant="alertType">
            {{ alertMessage }}
        </BootstrapAlert>

        <form @submit="saveMqttConfig">
            <div class="card">
                <div class="card-header text-bg-primary">MqTT Configuration</div>
                <div class="card-body">

                    <InputElement label="Enable MqTT"
                                  v-model="mqttConfigList.mqtt_enabled"
                                  type="checkbox" wide/>

                    <InputElement v-show="mqttConfigList.mqtt_enabled"
                                  label="Enable Home Assistant MQTT Auto Discovery"
                                  v-model="mqttConfigList.mqtt_hass_enabled"
                                  type="checkbox" wide/>

                </div>
            </div>

            <div class="card mt-5" v-show="mqttConfigList.mqtt_enabled">
                <div class="card-header text-bg-primary">
                    MqTT Broker Parameter
                </div>
                <div class="card-body">

                    <InputElement label="Hostname"
                                  v-model="mqttConfigList.mqtt_hostname"
                                  type="text" maxlength="128"
                                  placeholder="Hostname or IP address"/>

                    <InputElement label="Port"
                                  v-model="mqttConfigList.mqtt_port"
                                  type="number" min="1" max="65535"
                                  placeholder="Port number"/>

                    <InputElement label="Username"
                                  v-model="mqttConfigList.mqtt_username"
                                  type="text" maxlength="32"
                                  placeholder="Username, leave empty for anonymous connection"/>

                    <InputElement label="Password"
                                  v-model="mqttConfigList.mqtt_password"
                                  type="password" maxlength="32"
                                  placeholder="Password, leave empty for anonymous connection"/>

                    <InputElement label="Base Topic"
                                  v-model="mqttConfigList.mqtt_topic"
                                  type="text" maxlength="32"
                                  placeholder="Base topic, will be prepend to all published topics (e.g. inverter/)"/>

                    <InputElement label="Publish Interval"
                                  v-model="mqttConfigList.mqtt_publish_interval"
                                  type="number" min="5" max="86400" postfix="seconds"
                                  placeholder="Publish Interval in Seconds"/>

                    <InputElement label="Enable Retain Flag"
                                  v-model="mqttConfigList.mqtt_retain"
                                  type="checkbox"/>

                    <InputElement label="Enable TLS"
                                  v-model="mqttConfigList.mqtt_tls"
                                  type="checkbox"/>

                    <InputElement v-show="mqttConfigList.mqtt_tls"
                                  label="CA-Root-Certificate (default Letsencrypt)"
                                  v-model="mqttConfigList.mqtt_root_ca_cert"
                                  type="textarea" maxlength="2048" rows="10"
                                  placeholder="Root CA Certificate from Letsencrypt"/>
                </div>
            </div>

            <div class="card mt-5" v-show="mqttConfigList.mqtt_enabled">
                <div class="card-header text-bg-primary">LWT Parameters</div>
                <div class="card-body">

                    <InputElement label="LWT Topic"
                                  v-model="mqttConfigList.mqtt_lwt_topic"
                                  type="text" maxlength="32" :prefix="mqttConfigList.mqtt_topic"
                                  placeholder="LWT topic, will be append base topic"/>

                    <InputElement label="LWT Online message"
                                  v-model="mqttConfigList.mqtt_lwt_online"
                                  type="text" maxlength="20"
                                  placeholder="Message that will be published to LWT topic when online"/>

                    <InputElement label="LWT Offline message"
                                  v-model="mqttConfigList.mqtt_lwt_offline"
                                  type="text" maxlength="20"
                                  placeholder="Message that will be published to LWT topic when offline"/>
                </div>
            </div>

            <div class="card mt-5" v-show="mqttConfigList.mqtt_enabled && mqttConfigList.mqtt_hass_enabled">
                <div class="card-header text-bg-primary">Home Assistant MQTT Auto Discovery Parameters</div>
                <div class="card-body">

                    <InputElement label="Prefix Topic"
                                  v-model="mqttConfigList.mqtt_hass_topic"
                                  type="text" maxlength="32"
                                  placeholder="The prefix for the discovery topic"/>

                    <InputElement label="Enable Retain Flag"
                                  placeholder="hass retain"
                                  v-model="mqttConfigList.mqtt_hass_retain"
                                  type="checkbox"/>

                    <InputElement label="Enable Expiration"
                                  v-model="mqttConfigList.mqtt_hass_expire"
                                  type="checkbox"/>

                    <InputElement label="Individual Panels"
                                  v-model="mqttConfigList.mqtt_hass_individualpanels"
                                  type="checkbox"/>

                </div>
            </div>

            <button type="submit" class="btn btn-primary mb-3">Save</button>
        </form>
    </BasePage>
</template>

<script lang="ts">
import {defineComponent} from 'vue';
import BasePage from '@/components/BasePage.vue';
import BootstrapAlert from '@/components/BootstrapAlert.vue';
import InputElement from '@/components/InputElement.vue';
import {handleResponse, authHeader} from '@/utils/authentication';
import type {MqttConfig} from '@/types/MqttConfig';

export default defineComponent({
    components: {
        BasePage,
        BootstrapAlert,
        InputElement
    },
    data() {
        return {
            dataLoading: true,
            mqttConfigList: {} as MqttConfig,
            alertMessage: '',
            alertType: 'info',
            showAlert: false,
        };
    },
    created() {
        this.getMqttConfig();
    },
    methods: {
        getMqttConfig() {
            this.dataLoading = true;
            fetch('/api/mqtt/config', {headers: authHeader()})
                .then((response) => handleResponse(response, this.$emitter))
                .then((data) => {
                    this.mqttConfigList = data;
                    this.dataLoading = false;
                });
        },
        saveMqttConfig(e: Event) {
            e.preventDefault();

            const formData = new FormData();
            formData.append('data', JSON.stringify(this.mqttConfigList));

            fetch('/api/mqtt/config', {
                method: 'POST',
                headers: authHeader(),
                body: formData,
            })
                .then((response) => handleResponse(response, this.$emitter))
                .then(
                    (response) => {
                        this.alertMessage = response.message;
                        this.alertType = response.type;
                        this.showAlert = true;
                    }
                );
        },
    },
});
</script>
