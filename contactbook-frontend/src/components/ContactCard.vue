<script>
export default {
    props: {
        contact: { type: Object, required: true },
    },
};
</script>
<template>
    <div>
        <div class="p-1">
            <strong>Tên:</strong>
            {{ contact.name }}
        </div>
        <div class="p-1">
            <strong>E-mail:</strong>
            {{ contact.email }}
        </div>
        <div class="p-1">
            <strong>Địa chỉ:</strong>
            {{ contact.address }}
        </div>
        <div class="p-1">
            <strong>Điện thoại:</strong>
            {{ contact.phone }}
        </div>
        <div class="p-1">
            <strong>Liên hệ yêu thích:&nbsp;</strong>
            <i v-if="contact.favorite" class="fas fa-check"></i>
            <i v-else class="fas fa-times"></i>
        </div>

        <div v-if="contact.favorite && contact.parentCategory" class="p-1 border-top mt-2 pt-2">
            <div>
                <strong>Thể loại chính:</strong> {{ contact.parentCategory }}
            </div>
            <div v-if="contact.subCategories && contact.subCategories.length > 0" class="mt-1">
                <strong>Chi tiết:</strong>
                <span v-for="(sub, index) in contact.subCategories" :key="index" class="badge bg-secondary mr-1 ml-1">
                    {{ sub }}
                </span>
            </div>
        </div>
    </div>
</template>