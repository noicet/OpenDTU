<template>
    <BasePage :title="'Network Settings'" :isLoading="dataLoading">
        <BootstrapAlert v-model="showAlert" dismissible :variant="alertType">
            {{ alertMessage }}
        </BootstrapAlert>

        <form @submit="saveNetworkConfig">
            <div class="card">
                <div class="card-header text-bg-primary">WiFi Configuration</div>
                <div class="card-body">
                    <InputElement label="WiFi SSID"
                                  v-model="networkConfigList.ssid"
                                  type="text" maxlength="32"
                                  placeholder="SSID"/>

                    <InputElement label="WiFi Password"
                                  v-model="networkConfigList.password"
                                  type="password" maxlength="64"
                                  placeholder="PSK"/>

                    <InputElement label="Hostname"
                                  v-model="networkConfigList.hostname"
                                  type="text" maxlength="32"
                                  placeholder="SSID"
                    >
                        <div class="alert alert-secondary" role="alert">
                            <b>Hint:</b> The text <span class="font-monospace">%06X</span> will be replaced
                            with the last 6 digits of the ESP ChipID in hex format.
                        </div>
                    </InputElement>

                    <InputElement label="Enable DHCP"
                                  v-model="networkConfigList.dhcp"
                                  type="checkbox"/>
                </div>
            </div>

            <div class="card" v-show="!networkConfigList.dhcp">
                <div class="card-header text-bg-primary">
                    Static IP Configuration
                </div>
                <div class="card-body">
                    <InputElement label="IP Address"
                                  v-model="networkConfigList.ipaddress"
                                  type="text" maxlength="32"
                                  placeholder="IP address"/>

                    <InputElement label="Netmask"
                                  v-model="networkConfigList.netmask"
                                  type="text" maxlength="32"
                                  placeholder="Netmask"/>

                    <InputElement label="Default Gateway"
                                  v-model="networkConfigList.gateway"
                                  type="text" maxlength="32"
                                  placeholder="Default Gateway"/>

                    <InputElement label="DNS Server 1"
                                  v-model="networkConfigList.dns1"
                                  type="text" maxlength="32"
                                  placeholder="DNS Server 1"/>

                    <InputElement label="DNS Server 2"
                                  v-model="networkConfigList.dns2"
                                  type="text" maxlength="32"
                                  placeholder="DNS Server 2"/>

                </div>
            </div>
            <button type="submit" class="btn btn-primary mb-3">Save</button>
        </form>
    </BasePage>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import BasePage from '@/components/BasePage.vue';
import BootstrapAlert from "@/components/BootstrapAlert.vue";
import InputElement from '@/components/InputElement.vue';
import { handleResponse, authHeader } from '@/utils/authentication';
import type { NetworkConfig } from "@/types/NetworkkConfig";

export default defineComponent({
    components: {
        BasePage,
        BootstrapAlert,
        InputElement
    },
    data() {
        return {
            dataLoading: true,
            networkConfigList: {} as NetworkConfig,
            alertMessage: "",
            alertType: "info",
            showAlert: false,
        };
    },
    created() {
        this.getNetworkConfig();
    },
    methods: {
        getNetworkConfig() {
            this.dataLoading = true;
            fetch("/api/network/config", { headers: authHeader() })
                .then((response) => handleResponse(response, this.$emitter))
                .then((data) => {
                    this.networkConfigList = data;
                    this.dataLoading = false;
                });
        },
        saveNetworkConfig(e: Event) {
            e.preventDefault();

            const formData = new FormData();
            formData.append("data", JSON.stringify(this.networkConfigList));

            fetch("/api/network/config", {
                method: "POST",
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