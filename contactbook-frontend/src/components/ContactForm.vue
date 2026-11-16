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
            <Field name="phone" type="text" class="form-control" v-model="contactLocal.phone" />
            <ErrorMessage name="phone" class="error-feedback" />
        </div>

        <!-- Checkbox Liên hệ yêu thích -->
        <div class="form-group form-check">
            <input name="favorite" type="checkbox" class="form-check-input" v-model="contactLocal.favorite"
                id="favorite" />
            <label for="favorite" class="form-check-label">
                <strong>Liên hệ yêu thích</strong>
            </label>
        </div>

        <!-- Phần xổ ra khi tích chọn Liên hệ yêu thích -->
        <div v-if="contactLocal.favorite" class="mt-3 p-3 border rounded bg-light">
            <label class="form-label font-weight-bold">Chọn thể loại chính (Cha):</label>
            <div class="d-flex gap-4 mb-3">
                <div v-for="(cat, index) in categories" :key="index" class="form-check form-check-inline">
                    <input class="form-check-input" type="radio" name="parentCategory" :id="'cat_' + index"
                        :value="cat.name" v-model="contactLocal.parentCategory"
                        @change="contactLocal.subCategories = []" />
                    <label class="form-check-label" :for="'cat_' + index">{{ cat.name }}</label>
                </div>
            </div>

            <!-- Phần con tương ứng khi đã chọn danh mục cha -->
            <div v-if="contactLocal.parentCategory" class="mt-2">
                <label class="form-label font-weight-bold">Chọn chi tiết (Con) cho "{{ contactLocal.parentCategory
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
            <button v-if="contactLocal._id" type="button" class="ml-2 btn btn-danger"
                @click="deleteContact">Xóa</button>
            <button type="button" class="ml-2 btn btn-secondary" @click="onCancel">Thoát</button>
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
    props: {
        contact: { type: Object, required: true }
    },
    emits: ["submit:contact", "delete:contact"],
    data() {
        const contactFormSchema = yup.object().shape({
            name: yup
                .string()
                .required("Tên phải có giá trị.")
                .min(2, "Tên phải ít nhất 2 ký tự.")
                .max(50, "Tên có nhiều nhất 50 ký tự."),
            email: yup.string().email("E-mail không đúng.").max(50, "E-mail tối đa 50 ký tự."),
            address: yup.string().max(100, "Địa chỉ tối đa 100 ký tự."),
            phone: yup
                .string()
                .matches(
                    /((09|03|07|08|05)+([0-9]{8})\b)/g,
                    "Số điện thoại không hợp lệ."
                ),
        });

        return {
            // Khởi tạo dữ liệu bám sát prop truyền vào và thêm các trường danh mục
            contactLocal: {
                ...this.contact,
                favorite: this.contact.favorite || false,
                parentCategory: this.contact.parentCategory || "",
                subCategories: this.contact.subCategories || [],
            },
            // Định nghĩa danh mục cha và các mục con tương ứng
            categories: [
                { name: "Thể thao", children: ["Bóng đá", "Bắn súng", "Cầu lông"] },
                { name: "Âm nhạc", children: ["Pop", "Rock", "Ballad"] },
                { name: "Tin tức", children: ["Thời sự", "Công nghệ", "Giải trí"] }
            ],
            contactFormSchema,
        };
    },
    computed: {
        // Lọc danh sách con dựa trên danh mục cha đang được chọn
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
            this.$emit("delete:contact", this.contactLocal._id);
        },
        onCancel() {
            this.$router.push({ name: "contactbook" });
        },
    },
};
</script>

<style scoped>
@import "@/assets/form.css";

.error-feedback {
    color: red;
}
</style>