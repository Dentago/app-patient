<template>
    <div id="navbar">

        <button v-if="$route.path !== '/login' && $route.path !== '/register'" class="btn btn-link" @click="navigateTo('/')"
            data-toggle="tooltip" data-placement="bottom" title="Home">
            <span class="material-symbols-outlined">
                home
            </span>
        </button>

        <div v-if="$route.path !== '/login' && $route.path !== '/register'" class="dropdown">
            <button class="btn dropdown-toggle" id="notificationsDropdown" data-bs-toggle="dropdown" aria-expanded="false">
                <span v-if="notifications.length === 0" class="material-symbols-outlined">
                    notifications
                </span>
                <span v-else class="material-symbols-outlined">
                    notifications_unread
                </span>
            </button>
            <div class="dropdown-menu dropdown-menu-end p-0" aria-labelledby="notificationsDropdown" @click="getNotifications">
                <div class="list-group">
                    <h5 v-if="notifications.length === 0">No notifications</h5>
                    <NotificationItem v-for="notification in notifications" :key="notification._id"
                        :category="notification.category" :message="notification.message"
                        :timeslots="notification.timeslots">
                    </NotificationItem>
                </div>
            </div>
        </div>

        <button v-if="$route.path !== '/login' && $route.path !== '/register'" class="btn btn-link" @click="redirect()"
            data-toggle="tooltip" data-placement="bottom" title="Profile">
            <span class="material-symbols-outlined">
                account_circle
            </span>
        </button>

        <button v-if="$route.path !== '/login' && $route.path !== '/register'" class="btn btn-link" @click="handleLogout()"
            data-toggle="tooltip" data-placement="bottom" title="Logout">
            <span class="material-symbols-outlined">
                logout
            </span>
        </button>

        <h2 v-if="$route.path === '/login' || $route.path === '/register'">
            Welcome to dentago!
        </h2>
    </div>
</template>


<style scoped>
#navbar {
    display: flex;
    box-shadow: none;
    background-color: #ffa686;
    justify-content: space-evenly;
    padding: 1%;
    position: sticky;
    top: 0;
    left: 0;
    right: 0;
    z-index: 1;
}

#title {
    font-size: larger;
}

.dropdown {
    position: relative;
    display: inline-block;
}

.btn {
    background-color: #ff865a;
    padding: 12px;
    font-size: medium;
    /* border: none; */
    cursor: pointer;
}


.dropdown-menu {
    background-color: beige;
    width: 400px;
    height: 300px;
    overflow-x: hidden;
    overflow-y: auto;
}

.list-group {
    padding: 10px;
}

.material-symbols-outlined {
    color: black;
    font-size: 1.8em;
    /* Adjust the size as needed */
    /* transition: transform 0.2s, color 0.2s; */
}

.material-symbols-outlined:hover {
    /* transform: scale(1.2); Adjust the scale factor as needed for the pop effect */
    color: #333;
    /* Darker shade of black for hover effect */
}
</style>

<script>
import NotificationItem from '@/components/NotificationItem.vue'
import { Api } from '../Api'

export default {
    data() {
        return {
            // TODO: remove placeholder values
            accessToken: localStorage.getItem("access-token"),
            notifications: [
                { _id: 1, category: "CANCEL", message: "Appointment on Sunday, December 10th 2023 has been cancelled by dentist John.", timeslots: ["123rasd21dsad"] },
                { _id: 2, category: "CANCEL", message: "Appointment on Monday, December 11th 2023 has been cancelled by dentist John.", timeslots: ["123rasd21dsad"] },
                { _id: 3, category: "RECOMMENDATION", message: "The following appointments have been recommended to you: ", timeslots: ["123rasd21dsad", "12dsa2sasa"] },
            ]
        }
    },
    components: {
        NotificationItem
    },
    mounted() {
        window.addEventListener('route-change', this.getNotifications);
    },
    created() {
        this.getNotifications(),
        setInterval(this.getNotifications, 5000); // refresh the timeslot data every minute (60 000 milliseconds)
    },
    methods: {
        navigateTo(route) {
            this.$router.push(route);
        },
        redirect() {
            const username = JSON.parse(localStorage.getItem("patientData")).id;
            this.$router.push('/user/' + username);
        },
        async getNotifications() {
            if (!localStorage.getItem("patientData"))
            {
                this.notifications = [];
                return;
            }
            const userId = JSON.parse(localStorage.getItem("patientData")).id;
            Api.get(`/patients/${userId}/notifications`, {headers: {Authorization: `Bearer ${localStorage.getItem("access-token")}`}},
            {
                id: userId,
            })
                .then((res) => {
                    this.notifications = res.data;
                })
                .catch((err) => {
                    console.log(err)
                })
        },
        async handleLogout() {
            const userId = JSON.parse(localStorage.getItem("patientData")).id;
            console.log(userId);
            Api.patch('/logout', {
                id: userId,
            })
                .then((res) => {
                    console.log(res);
                    localStorage.removeItem("patientData");
                    localStorage.removeItem("access-token");
                    this.$router.push('/login')
                })
                .catch((err) => {
                    console.log(err)
                })
        }
    }
}
</script>
