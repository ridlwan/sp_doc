<template>
    <Layout>
        <template #heading>
            <h6 class="font-weight-bolder text-white mb-0">Attendance</h6>
        </template>

        <div class="row" v-if="status == 'Active'">
            <div class="col-lg-12 col-md-12">
                <div class="card">
                    <div class="card-body">
                        <div class="row">
                            <div class="col-md-10 mt-2">
                                <p class="text-uppercase">Your activity</p>
                            </div>
                            <div class="col-md-2 text-end">
                                <a href="javascript:;" @click="checkOut" class="btn btn-icon bg-gradient-danger d-lg-block mt-0 mb-0">
                                    Check Out
                                    <i class="fas fa-briefcase ms-1"></i>
                                </a>
                            </div>
                        </div>
                        <hr class="horizontal dark" />
                        <div id="activities" style="overflow: auto; height: 300px">
                            <div v-for="(activity, activity_index) in activities" :key="activity.id">
                                <div class="d-flex">
                                    <span class="mb-0 me-3 font-weight-bolder opacity-7" style="white-space: nowrap">{{ activity.start_time }}</span>
                                    <div>
                                        <h6 class="mb-0">
                                            <small class="font-weight-normal">{{ activity.description }}</small>
                                        </h6>
                                        <button v-if="activity_index == activities.length - 1 && activity.struggle_text == 'No' && !activity.end" type="button" @click="struggling(activity.id)" class="btn btn-sm btn-reddit btn-icon-only rounded-circle mb-0" data-bs-toggle="tooltip" title="I'm struggling here">
                                            <span class="btn-inner--icon"><i class="fa-solid fa-user-ninja"></i></span>
                                        </button>

                                        <span v-if="activity.struggle_text == 'Yes'" class="badge bg-gradient-danger" style="text-transform: unset"><i class="fa-solid fa-user-ninja"></i> I'm struggling here</span>
                                    </div>
                                </div>
                                <hr class="horizontal dark mb-4" />
                            </div>
                        </div>
                        <div class="row" v-if="addActivityForm">
                            <div v-if="activities.length < 1 || relogin" class="row" style="padding: 0; margin: 0">
                                <div class="col-md-10">
                                    <div class="form-group">
                                        <input class="form-control" type="text" placeholder="Write your activity" v-model="form.description" :class="{ 'is-invalid': form.errors.description }" />
                                    </div>
                                </div>
                                <div class="col-md-2">
                                    <a href="javascript:;" class="btn btn-primary d-lg-block" @click="addActivity"><i class="fa-solid fa-play"></i>&nbsp; Start Activity</a>
                                </div>
                            </div>
                            <div v-else class="row" style="padding: 0; margin: 0">
                                <div class="col-md-8">
                                    <div class="form-group">
                                        <input class="form-control" type="text" placeholder="Write your activity" v-model="form.description" :class="{ 'is-invalid': form.errors.description }" />
                                    </div>
                                </div>
                                <div class="col-md-2">
                                    <div class="form-group">
                                        <Datepicker time-picker class="form-control" type="time" id="timepicker" v-model="startTime" :class="{ 'is-invalid': form.errors.start }" />
                                    </div>
                                </div>
                                <div class="col-md-2">
                                    <a href="javascript:;" class="btn btn-primary d-lg-block" @click="addActivity"><i class="fa-solid fa-play"></i>&nbsp; Start Activity</a>
                                </div>
                            </div>

                            <div class="text-center">
                                <a href="javascript:;" @click="closeActivityForm"
                                    ><span class="text-secondary">Hold on a second <i class="fa-solid fa-person-digging ms-1"></i></span
                                ></a>
                            </div>
                        </div>
                        <div v-else class="text-center">
                            <button type="submit" class="btn btn-success" @click="openActivityForm"><i class="fa-solid fa-plus"></i>&nbsp; Add Activity</button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="row" v-else>
            <div class="col-lg-5 col-md-5 mt-4 mt-sm-0">
                <div class="card move-on-hover">
                    <div class="card-body">
                        <div class="w-20 mx-auto">
                            <img src="/assets/img/idea.png" alt="" class="img-fluid" />
                        </div>
                        <h6 class="card-description mt-4" v-html="quote"></h6>
                        <div class="row text-center">
                            <button @click="checkIn" class="btn btn-icon bg-gradient-primary d-lg-block mt-5 mb-3">
                                Check In
                                <i class="fas fa-laptop-code ms-1"></i>
                            </button>
                        </div>
                        <hr class="horizontal dark" />
                        <div class="row">
                            <div class="col-md-6">
                                <a href="javascript:;" @click="outOfOffice" class="btn btn-icon bg-gradient-warning d-lg-block mt-3 mb-0">
                                    Out of Office
                                    <i class="fas fa-snowboarding ms-1"></i>
                                </a>
                            </div>
                            <div class="col-md-6 text-end">
                                <a href="javascript:;" @click="outSick" class="btn btn-icon bg-gradient-danger d-lg-block mt-3 mb-0">
                                    Out Sick
                                    <i class="fas fa-head-side-cough ms-1"></i>
                                </a>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </Layout>
</template>

<style>
.dp__main {
    padding: 0px !important;
}
.dp__input_wrap {
    border: unset !important;
    padding-top: 1px;
    padding-bottom: 1.5px;
}
.dp__input {
    border: unset;
    border-radius: 8px;
    width: 115px;
}
</style>

<script setup>
import $ from "jquery";
import { Inertia } from "@inertiajs/inertia";
import { ref, onMounted, computed } from "vue";
import { useForm, usePage, Link } from "@inertiajs/inertia-vue3";
import Layout from "../../Components/LayoutAttendance.vue";
import Swal from "sweetalert2";
import Datepicker from "@vuepic/vue-datepicker";
import "@vuepic/vue-datepicker/dist/main.css";

const form = useForm({
    description: null,
    start: null,
});

let addActivityForm = ref(false);
let startTime = ref(null);

const props = defineProps({
    quote: String,
    activities: Array,
    relogin: Boolean,
});

onMounted(() => {
    document.body.classList.add("virtual-reality");
    document.getElementById("navbar-main").classList.add("mt-3");
    document.getElementById("navbar-main").classList.add("mx-3");
    document.getElementById("navbar-main").classList.add("bg-primary");

    if (status == "Active" && props.activities.length > 0) {
        scrollDown();
    }
});

const status = computed(() => usePage().props.value.auth.user.status);

const startTimeData = computed(() => {
    if (startTime.value) {
        form.start = startTime.value.hours + ":" + startTime.value.minutes;
    } else {
        form.start = null;
    }
});

const checkIn = () => {
    if (props.relogin) {
        Swal.fire({
            title: "Fight longer? <br> <i class='fa-solid fa-robot'></i>",
            text: "Do you want to re-login?",
            icon: "warning",
            showCancelButton: true,
            cancelButtonColor: "orange",
            confirmButtonText: "Let's Go! <i class='fa-solid fa-person-skating'></i>",
            cancelButtonText: "Nope, Just kidding <i class='fa-solid fa-bed'></i>",
        }).then((result) => {
            if (result.isConfirmed) {
                Inertia.post("/attendance/check-in");
                Swal.fire({
                    title: "Welcome Back <br> <i class='fa-solid fa-user-secret'></i>",
                    text: "Continue unfinished mission",
                    icon: "success",
                    confirmButtonText: "Sure",
                });
            }
        });
    } else {
        Inertia.post("/attendance/check-in");
        Swal.fire({
            title: "You're in! <br> <i class='fa-solid fa-user-secret'></i>",
            text: "Start the journey and enjoy your work",
            icon: "success",
            confirmButtonText: "Cool",
        });
    }
};

const openActivityForm = () => {
    addActivityForm.value = true;
    form.errors = [];
    startTime.value = {
        hours: new Date().getHours(),
        minutes: new Date().getMinutes(),
    };
    form.start = startTime.value.hours + ":" + startTime.value.minutes;
};

const closeActivityForm = () => {
    addActivityForm.value = false;
};

const addActivity = () => {
    form.post("/attendance", {
        onSuccess: () => {
            addActivityForm.value = false;
            form.description = null;
            form.start = null;
            scrollDown();
            Swal.fire({
                icon: "success",
                title: "Activity added <br> <i class='fa-solid fa-user-astronaut'></i>",
                text: "Keep strong and continue your journey",
                showConfirmButton: false,
                timer: 3000,
            });
        },
    });
};

const checkOut = () => {
    if (props.activities.length < 1 || props.relogin) {
        Swal.fire({
            title: "What are you doing? <br> <i class='fa-solid fa-person-through-window'></i>",
            text: "Please write down your activity before checking out",
            icon: "error",
            confirmButtonText: "Got it <i class='fa-solid fa-pen-to-square'></i>",
        });
    } else {
        Swal.fire({
            title: "Are you sure? <br> <i class='fa-solid fa-door-open'></i>",
            text: "Do you want to check out now?",
            icon: "warning",
            showCancelButton: true,
            cancelButtonColor: "orange",
            confirmButtonText: "Yes, please! <i class='fa-solid fa-person-walking-dashed-line-arrow-right'></i>",
            cancelButtonText: "Not yet <i class='fa-solid fa-person-digging'></i>",
        }).then((result) => {
            if (result.isConfirmed) {
                Inertia.post("/attendance/check-out");
                Swal.fire({
                    title: "You're out! <br> <i class='fa-solid fa-child-reaching'></i>",
                    text: "Thanks for your great work today",
                    icon: "success",
                    confirmButtonText: "See ya <i class='fa-solid fa-hand'></i>",
                });
            }
        });
    }
};

const scrollDown = () => {
    $("#activities")
        .stop()
        .animate(
            {
                scrollTop: $("#activities")[0].scrollHeight,
            },
            800
        );
};

const struggling = (id) => {
    Swal.fire({
        title: "Whoops <br> <i class='fa-solid fa-user-ninja'></i>",
        text: "Are you really struggling here?",
        icon: "question",
        showCancelButton: true,
        confirmButtonColor: "red",
        cancelButtonColor: "orange",
        confirmButtonText: "Definitely! <i class='fa-solid fa-face-dizzy'></i></i>",
        cancelButtonText: "Nope, it just needs more time <i class='fa-solid fa-person-digging'></i>",
    }).then((result) => {
        if (result.isConfirmed) {
            Inertia.post("/attendance/struggle/" + id);
            Swal.fire({
                title: "It's okay <br> <i class='fa-solid fa-user-ninja'></i>",
                text: "Just take a deep breath. We know you will crush it",
                icon: "success",
                confirmButtonText: "Sure <i class='fa-solid fa-paw'></i>",
            });
        }
    });
};

const outOfOffice = () => {
    Inertia.post("/attendance/out-of-office");
};

const outSick = () => {
    Inertia.post("/attendance/out-sick");
};
</script>
