<template>
    <v-container>

        <v-row class="justify-space-between">
            <v-col cols="6" sm="6" md="6" lg="6" xl="6">
                <h1>Templates</h1>
            </v-col>
        </v-row>
        <v-row>
            <v-col cols="12" sm="12" md="3"
            v-for="template in services.services" :key="template.name">
                <v-card
                    style="padding-bottom: 40px;"
                    height="320px"
                    color="cardBackground">
                    <v-list-item-content class="justify-center">

                        <v-avatar
                            size="57"
                            rounded
                            ><img
                            :src="template.icon"
                            :alt="template.name"
                            ></v-avatar>
                        <v-card-title>
                            <a :href="template.website" target="_blank">{{ template.name }}</a>
                        </v-card-title>

                        <v-card-subtitle>
                            <v-chip
                                label
                                small
                                class="mr-2"
                            ><v-icon left small>mdi-star</v-icon>{{ template.stars }}</v-chip>
                            <v-chip
                                label
                                small
                                v-if="template.spdx_id && template.spdx_id !== 'NOASSERTION'"
                            ><v-icon left small>mdi-file-certificate</v-icon>{{ template.spdx_id }}</v-chip>
                        </v-card-subtitle>

                        <v-card-text>
                            {{ template.description }}
                            <!--Operator: <a :href="template.url">{{ template.id }}</a>-->
                        </v-card-text>
                    </v-list-item-content>
                </v-card>
                <div class="text-center" style="height:0px" v-if="!template.enabled">
                <v-btn
                    style="top: -50px;"
                    color="primary"
                    dark
                    @click="openInstallDialog(template)"
                    >
                    details
                </v-btn>
                </div>
            </v-col>
        </v-row>
        <v-dialog
            v-model="dialog"
            max-width="890"
            >
            <v-card>
                <v-card-title class="text-h5">
                    {{clickedTemplate.name}}
                </v-card-title>
                <v-card-text>
                    {{clickedTemplate.description}}
                </v-card-text>
                <v-card-text v-if="clickedTemplate.screenshots && clickedTemplate.screenshots.length > 0">
                    <v-carousel>
                        <v-carousel-item
                        v-for="(screenshot, i) in clickedTemplate.screenshots"
                        :key="i"
                        :src="screenshot"
                        >
                        </v-carousel-item>
                    </v-carousel>
                </v-card-text>
                <v-card-text>
                    <v-select
                        :items="pipelines"
                        label="Pipeline"
                        v-model="pipeline"
                    ></v-select>
                    <v-select
                        :items="phases[pipeline]"
                        label="Phase"
                        v-model="phase"
                    ></v-select>
                    <v-btn
                        color="primary"
                        dark
                        :disabled="!pipeline || !phase"
                        @click="openInstall(clickedTemplate.dirname, pipeline, phase)"
                        >
                        Install
                    </v-btn>
                </v-card-text>
            </v-card>
        </v-dialog>

    </v-container>
</template>

<script>
import axios from "axios";
export default {
    sockets: {
    },
    mounted() {
        this.loadTemplatesList();
        this.loadPipelinesList();
    },
    data: () => ({
        pipeline: undefined,
        phase: undefined,
        pipelines: [],
        phases: {},
        services: [],
        dialog: false,
        clickedTemplate: {},
    }),
    components: {
    },
    methods: {
        async copyInstall(install) {
            try {
                await navigator.clipboard.writeText(install);
            } catch($e) {
                console.log('Cannot copy');
            }
            this.dialog = false;
        },
        openInstall(templatename, pipeline, phase) {
            // redirect to install page
            console.log(`/#/pipeline/${pipeline}/${phase}/apps/new?service=${templatename}`);
            window.location.href = `/#/pipeline/${pipeline}/${phase}/apps/new?service=${templatename}`;

        },
        openInstallDialog(template) {
            this.clickedTemplate = template;
            this.dialog = true;
        },
        loadTemplatesList() {
            const self = this;
            axios.get(`https://services.kubero.dev`)
            .then(response => {
                self.services = response.data;
            })
            .catch(error => {
                console.log(error);
            });
        },
        loadPipelinesList() {
            const self = this;
            axios.get(`/api/pipelines`)
            .then(response => {
                for (let i = 0; i < response.data.items.length; i++) {
                    const pipeline = response.data.items[i];
                    let p = [];
                    for (let j = 0; j < pipeline.phases.length; j++) {
                        const phase = pipeline.phases[j];
                        if (phase.enabled == true) {
                            p.push(phase.name);
                        }
                    }
                    self.pipelines.push(pipeline.name);
                    self.phases[pipeline.name] = p;

                }
            })
            .catch(error => {
                console.log(error);
            });
        },
    },
}
</script>

<style lang="scss">
pre {
    background: #f4f4f4;
    border: 1px solid #ddd;
    border-left: 3px solid #f36d33;
    color: #666;
    page-break-inside: avoid;
    font-family: monospace;
    font-size: 15px;
    line-height: 1.6;
    margin-bottom: 1.6em;
    max-width: 100%;
    overflow: auto;
    padding: 1em 1.5em;
    display: block;
    word-wrap: break-word;
}
</style>