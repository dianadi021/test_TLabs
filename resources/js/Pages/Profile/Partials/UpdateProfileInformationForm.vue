<script setup>
import InputError from "@/Components/InputError.vue";
import InputLabel from "@/Components/InputLabel.vue";
import PrimaryButton from "@/Components/PrimaryButton.vue";
import TextInput from "@/Components/TextInput.vue";
import { Link, useForm, usePage } from "@inertiajs/vue3";

defineProps({
    mustVerifyEmail: {
        type: Boolean,
    },
    status: {
        type: String,
    },
});

const user = usePage().props.user_login;
console.log(user);

const form = useForm({
    fullname: user.fullname,
    birthdate: user.birthdate,
});

const submit = () => {
    const base_url = window.location.host;
    const [host, port] = base_url.split(":");
    const $base_url = isValidVal(port)
        ? `http://${host}:${port}`
        : `https://${host}`;

    Swal.fire({
        title: "Apakah kamu yakin ingin melanjutkan?",
        // text: "You won't be able to revert this!",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        cancelButtonText: "Batal",
        confirmButtonText: "Oke!",
    }).then((result) => {
        if (result.isConfirmed) {
            toastr.warning("Sedang diproses, mohon tunggu!", "Peringatan!");

            $("#btnRegister").hide();
            $("#loadingAjax").show();

            $.ajaxSetup({
                headers: {
                    "X-CSRF-TOKEN": $('meta[name="csrf-token"]').attr(
                        "content"
                    ),
                },
            });

            $("#_csrf-token").val($('meta[name="csrf-token"]').attr("content"));
            $("#csrf-token").val($('meta[name="csrf-token"]').attr("content"));

            $.ajax({
                url: `${$base_url}/profile`,
                type: "PATCH",
                data: $("#formBiodata").serializeArray(),
                xhrFields: {
                    withCredentials: true,
                },
                headers: {
                    "X-CSRF-TOKEN": $('meta[name="csrf-token"]').attr(
                        "content"
                    ),
                },
                success: function (callback) {
                    const { messages } = callback;
                    console.log("success", callback);
                    toastr.success(messages, "Success!");
                    $("#btnRegister").show();
                },
                error: function (callback) {
                    const { responseJSON } = callback;
                    const { errors, message, messages, datas } = responseJSON;
                    let errorInfo, validator;
                    if (datas) {
                        const { errorInfo: errInfo, validator: validCallback } =
                            datas;
                        errorInfo = errInfo;
                        validator = validCallback;
                    }
                    console.log("error", callback);

                    if (errors) {
                        for (let key in errors) {
                            toastr.error(errors[key][0], "Kesalahan!");
                            $(`#err_${key}`).show();
                            $(`#err_${key} li`).html(errors[key][0]);
                        }
                    } else if (message || messages || errorInfo || validator) {
                        const tmpMsg = validator
                            ? "input data tidak sesuai atau tidak boleh kosong"
                            : errorInfo
                            ? errorInfo[2]
                            : messages
                            ? messages
                            : message;
                        toastr.error(tmpMsg, "Kesalahan!");
                    }
                    $("#btnRegister").show();
                    $("#loadingAjax").hide();
                },
            });
        }
    });
};

$(document).ready(function () {
    $("#birthdate").datepicker({
        dateFormat: "dd/mm/yy",
    });
});
</script>

<template>
    <section>
        <header>
            <h2 class="text-lg font-medium text-gray-900">
                Profile Information
            </h2>

            <p class="mt-1 text-sm text-gray-600">
                Update your account's profile information and email address.
            </p>
        </header>

        <form id="formBiodata" @submit.prevent="submit" class="mt-6 space-y-6">
            <div>
                <InputLabel for="fullname" value="Fullname" />

                <TextInput
                    id="fullname"
                    name="fullname"
                    type="text"
                    class="mt-1 block w-full"
                    v-model="form.fullname"
                    autofocus
                    autocomplete="fullname"
                />

                <InputError class="mt-2" :message="form.errors.fullname" />
            </div>

            <div>
                <InputLabel for="birthdate" value="Date of Birth" />

                <TextInput
                    id="birthdate"
                    name="birthdate"
                    type="text"
                    class="mt-1 block w-full"
                    v-model="form.birthdate"
                    autofocus
                    autocomplete="birthdate"
                />

                <InputError class="mt-2" :message="form.errors.birthdate" />
            </div>

            <div class="flex items-center gap-4">
                <PrimaryButton :disabled="form.processing">Save</PrimaryButton>

                <Transition
                    enter-active-class="transition ease-in-out"
                    enter-from-class="opacity-0"
                    leave-active-class="transition ease-in-out"
                    leave-to-class="opacity-0"
                >
                    <p
                        v-if="form.recentlySuccessful"
                        class="text-sm text-gray-600"
                    >
                        Saved.
                    </p>
                </Transition>
            </div>
        </form>
    </section>
</template>
