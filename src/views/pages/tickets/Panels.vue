<template>
    <PageHeader title="Ticket Panels" description="Tickets panels allow your users to interact with the ticket system." />
    <NavCreate title="Create New Panel" icon="/images/svg/plus_sign.svg" :onClick="goToCreatePanel" />

    <div v-if="isLoading" class="flex justify-center items-center h-64">
        <CustomProgressSpinner />
    </div>

    <div v-else class="card">
        <PageHeader title="Your Ticket Panels" stats="1/25" />

        <div class="flex flex-col gap-4 mb-4">
            <label class="text-[#9090a3] font-bold">Default Message Footer</label>
            <CustomTextField customWidth="max-w-[120rem]" :isIcon="true" iconSrc="/images/svg/textField_icon.svg" v-model="textInput" placeholder="Enter a keyword (e.g., panel name, status, category)" />
        </div>

        <!-- Custom DataTable -->
        <div class="card mt-8 rounded-lg overflow-hidden">
            <DataTable :value="filteredTickets" tableStyle="min-width: 50rem" class="custom-datatable">
                <Column field="info" header="Info">
                    <template #body="slotProps">
                        <div class="flex items-center gap-4">
                            <div class="bg-[#1c1e2b] text-[1.4rem] rounded-md xl:w-[55px] xl:h-[55px] w-[40px] h-[40px] flex items-center justify-center">🎫</div>
                            <div>
                                <p class="xl:text-xl relative bottom-1 font-normal text-[#fff] text-lg">{{ slotProps.data.panelName }}</p>
                                <span class="text-md font-normal flex gap-1 items-center text-[#9f9fb4] text-base xl:text-sm"> <Img src="/images/svg/copy-sign.svg" />{{ slotProps.data.category }} </span>
                            </div>
                        </div>
                    </template>
                </Column>
                <Column field="status" header="Status">
                    <template #body="slotProps">
                        <span :class="{ 'text-green-500': slotProps.data.status === 'Active', 'text-red-500': slotProps.data.status === 'Inactive' }">
                            {{ slotProps.data.status }}
                        </span>
                    </template>
                </Column>
                <Column field="openTickets" header="Open Tickets">
                    <template #body="slotProps">
                        {{ slotProps.data.openTickets }}
                    </template>
                </Column>
                <Column>
                    <template #body="slotProps">
                        <div class="flex items-center">
                            <Button icon="pi pi-trash" size="large" @click="() => openDeleteModal(slotProps.data.id)" class="p-button-rounded p-button-danger p-button-text" />
                            <Button icon="pi pi-file-edit" size="large" @click="() => handleEditPanel(slotProps.data.id)" class="p-button-rounded p-button-text" />
                        </div>
                    </template>
                </Column>
            </DataTable>
        </div>
    </div>

    <CustomDeleteAlertModel ref="deleteModal" :onConfirm="handleDelete" confirmText="Delete Panel" />
    <Toast />
</template>

<script setup lang="ts">
import Button from 'primevue/button';
import Column from 'primevue/column';
import DataTable from 'primevue/datatable';
import { useToast } from 'primevue/usetoast';
import { computed, onMounted, ref } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import CustomDeleteAlertModel from '../../../components/CustomDeleteAlertModel.vue';
import CustomProgressSpinner from '../../../components/CustomProgressSpinner.vue';
import CustomTextField from '../../../components/CustomTextField.vue';
import NavCreate from '../../../components/NavCreate.vue';
import PageHeader from '../../../components/PageHeader.vue';
import { deletePanel, getPanelForGuilds } from '../../../service/panels.services';

const router = useRouter();
const toast = useToast();
const textInput = ref('');
const deleteModal = ref(null);
const isLoading = ref(true); // Loading state
const route = useRoute();
const id = route.params.id;
const selectedId = ref();
const openDeleteModal = (id) => {
    deleteModal.value.openModal();
    selectedId.value = id;
};

const tickets = ref([]);

const handleDelete = async () => {
    console.log(selectedId.value);
    if (selectedId.value) {
        try {
            tickets.value = tickets.value.filter((ticket) => ticket.id !== selectedId.value);
            await deletePanel(selectedId.value);
            toast.add({ severity: 'success', summary: 'Success', detail: 'Panel deleted successfully', life: 3000 });
            selectedId.value = null;
        } catch (error) {
            toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to delete panel', life: 3000 });
            console.error('Error deleting panel:', error);
        }
    }
};

const filteredTickets = computed(() => {
    const query = textInput.value.toLowerCase();
    if (!query) return tickets.value;

    return tickets.value.filter((ticket) => ticket.panelName.toLowerCase().includes(query) || ticket.status.toLowerCase().includes(query) || ticket.category.toLowerCase().includes(query));
});

function goToCreatePanel() {
    router.push(`/tickets/panels/create/${id}`);
}

function handleEditPanel(panelId) {
    router.push(`/tickets/panels/edit/${panelId}/${id}`);
}

onMounted(async () => {
    try {
        const data = await getPanelForGuilds(id);
        tickets.value = data.panels.map((panel) => {
            return {
                id: panel._id,
                panelName: panel.panelName,
                openTickets: 2,
                status: 'Active',
                category: data.cache.categories.find((category) => category.id == panel.ticketOpenCategoryId).name
            };
        });
    } catch (error) {
        console.error('Error fetching panels:', error);
    } finally {
        isLoading.value = false;
    }
});
</script>

<style scoped>
.custom-datatable ::v-deep(.p-datatable-wrapper) {
    background-color: #1e1e2d;
}

.custom-datatable ::v-deep(.p-datatable-tbody > tr) {
    color: white;
    border-bottom: 1px solid #2d2d3d;
}

.custom-datatable ::v-deep(.p-datatable-tbody > tr:last-child) {
    border-bottom: none;
}
</style>
