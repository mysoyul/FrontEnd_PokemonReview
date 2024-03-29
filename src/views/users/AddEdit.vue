<script setup>
import { Form, Field } from 'vee-validate';
import * as Yup from 'yup';
import { useRoute, useRouter } from 'vue-router';
import { storeToRefs } from 'pinia';

import { useUsersStore, useAlertStore } from '@/stores';

const usersStore = useUsersStore();
const alertStore = useAlertStore();

const router = useRouter();
const route = useRoute();
const id = route.params.id;

let title = 'Add User';
let user = null;
if (id) {
    console.log('Edit Mode = ' + id)
    // edit mode
    title = 'Edit User';
    usersStore.getById(id);
    ({ user } = storeToRefs(usersStore));
    console.log(user)
}

const schema = Yup.object().shape({
    firstName: Yup.string()
        .required('First Name is required'),
    lastName: Yup.string()
        .required('Last Name is required'),
    username: Yup.string()
        .required('Username is required'),
    password: Yup.string()
        .transform(x => x === '' ? undefined : x)
        // password optional in edit mode
        .concat(user ? null : Yup.string().required('Password is required'))
        .min(6, 'Password must be at least 6 characters'),
    role: Yup.string()
        .required('Role is required'),
});

async function onSubmit(values) {
    try {
        let message;
        if (user) {
            await usersStore.update(user.value.id, values)
            message = 'User updated';
        } else {
            await usersStore.register(values);
            message = 'User added';
        }
        await router.push('/users');
        alertStore.success(message);
    } catch (error) {
        alertStore.error(error);
    }
}
</script>

<template>
    <h1>{{title}}</h1>
    <template v-if="!(user?.loading || user?.error)">
        <Form @submit="onSubmit" :validation-schema="schema" :initial-values="user" v-slot="{ errors, isSubmitting }">
            <div class="form-row">
                <div class="form-group col">
                    <label>First Name</label>
                    <Field name="firstName" type="text" class="form-control" :class="{ 'is-invalid': errors.firstName }" />
                    <div class="invalid-feedback">{{ errors.firstName }}</div>
                </div>
                <div class="form-group col">
                    <label>Last Name</label>
                    <Field name="lastName" type="text" class="form-control" :class="{ 'is-invalid': errors.lastName }" />
                    <div class="invalid-feedback">{{ errors.lastName }}</div>
                </div>
            </div>
            <div class="form-row">
                <div class="form-group col">
                    <label>Username</label>
                    <Field name="username" type="text" class="form-control" :class="{ 'is-invalid': errors.username }" 
                        :disabled="user ? true : false"  />
                    <div class="invalid-feedback">{{ errors.username }}</div>
                </div>
                <div class="form-group col">
                    <label>
                        Password
                        <!-- <em v-if="user">(Leave blank to keep the same password)</em> -->
                    </label>
                    <Field name="password" type="password" class="form-control" :class="{ 'is-invalid': errors.password }" />
                    <div class="invalid-feedback">{{ errors.password }}</div>
                </div>
            </div>
            <div class="form-row">                
                <div class="form-group col">
                    <label>Role</label>
                    <div class="form-check">
                        <Field name="role" class="form-check-input" type="radio"  id="roleId1" value="ROLE_USER" :class="{ 'is-invalid': errors.role }" />
                        <label class="form-check-label" for="roleId1">ROLE_USER</label>
                        <div class="invalid-feedback">{{ errors.role }}</div>
                    </div>    
                </div>
                <div class="form-group col"> 
                    <label></label>
                    <div class="form-check">
                        <Field name="role" class="form-check-input" type="radio"  id="roleId2" value="ROLE_ADMIN" :class="{ 'is-invalid': errors.role }" />
                        <label class="form-check-label" for="roleId2">ROLE_ADMIN</label>
                        <div class="invalid-feedback">{{ errors.role }}</div>
                    </div>    
                </div>
                       
            </div>
            <div class="form-group">
                <button class="btn btn-primary" :disabled="isSubmitting">
                    <span v-show="isSubmitting" class="spinner-border spinner-border-sm mr-1"></span>
                    Save
                </button>
                <router-link to="/users" class="btn btn-link">Cancel</router-link>
            </div>
        </Form>
    </template>
    <template v-if="user?.loading">
        <div class="text-center m-5">
            <span class="spinner-border spinner-border-lg align-center"></span>
        </div>
    </template>
    <template v-if="user?.error">
        <div class="text-center m-5">
            <div class="text-danger">Error loading user: {{user.error}}</div>
        </div>
    </template>
</template>