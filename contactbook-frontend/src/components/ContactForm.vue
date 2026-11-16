<template>
    <Form @submit="submitContact" :validation-schema="contactFormSchema">
        <div class="form-group">
            <label for="name">Tên</label>
            <Field name="name" type="text" class="form-control" v-model="contactLocal.name" />
            <ErrorMessage name="name" class="error-feedback" />
        </div>
        <div class="form-group">
            <label for="email">E-mail</label>
            <Field name="email" type="email" class="form-control" v-model="contactLocal.email" />
            <ErrorMessage name="email" class="error-feedback" />
        </div>
        <div class="form-group">
            <label for="address">Địa chỉ</label>
            <Field name="address" type="text" class="form-control" v-model="contactLocal.address" />
            <ErrorMessage name="address" class="error-feedback" />
        </div>
        <div class="form-group">
            <label for="phone">Điện thoại</label>
            <Field name="phone" type="tel" class="form-control" v-model="contactLocal.phone" />
            <ErrorMessage name="phone" class="error-feedback" />
        </div>

        <div class="form-group form-check">
            <!-- Thêm @change để tự động làm sạch danh mục con nếu bỏ chọn yêu thích -->
            <input name="favorite" type="checkbox" class="form-check-input" v-model="contactLocal.favorite"
                id="favorite" @change="handleFavoriteChange" />
            <label for="favorite" class="form-check-label">
                <strong>Liên hệ yêu thích</strong>
            </label>
        </div>

        <div v-if="contactLocal.favorite" class="mt-3 p-3 border rounded bg-light">
            <label class="form-label font-weight-bold">Chọn thể loại chính:</label>
            <div class="d-flex gap-4 mb-3">
                <div v-for="(cat, index) in categories" :key="index" class="form-check form-check-inline">
                    <input class="form-check-input" type="radio" name="parentCategory" :id="'cat_' + index"
                        :value="cat.name" v-model="contactLocal.parentCategory"
                        @change="contactLocal.subCategories = []" />
                    <label class="form-check-label" :for="'cat_' + index">{{ cat.name }}</label>
                </div>
            </div>

            <div v-if="contactLocal.parentCategory" class="mt-2">
                <label class="form-label font-weight-bold">Chọn chi tiết cho "{{ contactLocal.parentCategory
                    }}":</label>
                <div class="d-flex flex-wrap gap-4">
                    <div v-for="(sub, subIndex) in currentSubCategories" :key="subIndex"
                        class="form-check form-check-inline">
                        <input class="form-check-input" type="checkbox" :id="'sub_' + subIndex" :value="sub"
                            v-model="contactLocal.subCategories" />
                        <label class="form-check-label" :for="'sub_' + subIndex">{{ sub }}</label>
                    </div>
                </div>
            </div>
        </div>

        <div class="form-group mt-3">
            <button class="btn btn-primary">Lưu</button>
            <button v-if="contactLocal._id" type="button" class="ml-2 btn btn-danger" @click="deleteContact">
                Xóa
            </button>
            <button type="button" class="ml-2 btn btn-danger" @click="Cancel">
                Thoát
            </button>
        </div>
    </Form>
</template>

<script>
import * as yup from "yup";
import { Form, Field, ErrorMessage } from "vee-validate";
export default {
    components: {
        Form,
        Field,
        ErrorMessage,
    },
    emits: ["submit:contact", "delete:contact"],
    props: {
        contact: { type: Object, required: true }
    },
    data() {
        const contactFormSchema = yup.object().shape({
            name: yup
                .string()
                .required("Tên phải có giá trị.")
                .min(2, "Tên phải ít nhất 2 ký tự.")
                .max(50, "Tên có nhiều nhất 50 ký tự."),
            email: yup
                .string()
                .email("E-mail không đúng.")
                .max(50, "E-mail tối đa 50 ký tự."),
            address: yup.string().max(100, "Địa chỉ tối đa 100 ký tự."),
            phone: yup
                .string()
                .matches(
                    /((09|03|07|08|05)+([0-9]{8})\b)/g,
                    "Số điện thoại không hợp lệ."
                ),
        });

        // Bổ sung các giá trị mặc định cho parentCategory và subCategories vào contactLocal sẵn có
        const contactLocal = {
            ...this.contact,
            parentCategory: this.contact.parentCategory || "",
            subCategories: this.contact.subCategories || [],
        };

        return {
            contactLocal,
            contactFormSchema,
            // Khai báo danh mục cha và các con tương ứng
            categories: [
                { name: "Thể thao", children: ["Bóng đá", "Bắn súng", "Cầu lông"] },
                { name: "Âm nhạc", children: ["Pop", "Rock", "Ballad"] },
                { name: "Tin tức", children: ["Thời sự", "Công nghệ", "Giải trí"] }
            ],
        };
    },
    computed: {
        // Lọc danh sách con dựa trên danh mục cha được chọn
        currentSubCategories() {
            const found = this.categories.find(c => c.name === this.contactLocal.parentCategory);
            return found ? found.children : [];
        }
    },
    methods: {
        submitContact() {
            this.$emit("submit:contact", this.contactLocal);
        },
        deleteContact() {
            this.$emit("delete:contact", this.contactLocal.id);
        },
        Cancel() {
            const reply = window.confirm('You have unsaved changes! Do you want to leave ? ')
            if (!reply) {
                return false
            }
            else this.$router.push({ name: "contactbook" });
        },
        // Hàm phụ trợ khi bỏ chọn yêu thích sẽ reset lại các danh mục đã chọn
        handleFavoriteChange() {
            if (!this.contactLocal.favorite) {
                this.contactLocal.parentCategory = "";
                this.contactLocal.subCategories = [];
            }
        }
    },
};
</script>
<style scoped>
@import "@/assets/form.css";
</style>