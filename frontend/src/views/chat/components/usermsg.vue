<template>
    <div class="user_msg_container" ref="containerRef">
        <!-- 显示@的知识库和文件 -->
        <div v-if="mentioned_items && mentioned_items.length > 0" class="mentioned_items">
            <span
                v-for="item in mentioned_items"
                :key="item.id"
                class="mentioned_tag"
                :class="[
                    item.type === 'kb'
                        ? item.kb_type === 'faq'
                            ? 'faq-tag'
                            : 'kb-tag'
                        : 'file-tag',
                ]">
                <span class="tag_icon">
                    <t-icon
                        v-if="item.type === 'kb'"
                        :name="item.kb_type === 'faq' ? 'chat-bubble-help' : 'folder'" />
                    <t-icon v-else name="file" />
                </span>
                <span class="tag_name">{{ item.name }}</span>
            </span>
        </div>
        <!-- 显示上传的图片 -->
        <div v-if="hasImages" class="user_images">
            <img
                v-for="(img, idx) in props.images"
                :key="idx"
                :src="img.url"
                class="user_image_thumb"
                @click="previewImage($event)" />
        </div>

        <!-- 显示非图片附件（文件、音频等） -->
        <div v-if="hasAttachments" class="user_attachments">
            <div v-for="(att, idx) in props.attachments" :key="idx" class="user_attachment_item">
                <a
                    :href="att.url"
                    :download="att.name"
                    class="attachment_link"
                    target="_blank"
                    rel="noopener noreferrer">
                    <t-icon :name="getIcon(att.type)" class="attachment_icon" />
                    <div class="attachment_meta">
                        <div class="attachment_name" :title="att.name">{{ att.name }}</div>
                        <div class="attachment_size">{{ formatBytes(att.size) }}</div>
                    </div>
                </a>
            </div>
        </div>
        <div class="user_msg">
            {{ content }}
        </div>
        <picturePreview :reviewImg="reviewImg" :reviewUrl="reviewUrl" @closePreImg="closePreImg" />
    </div>
</template>
<script setup>
import { defineProps, computed, ref, watch, onMounted, nextTick } from "vue"
import { hydrateProtectedFileImages } from "@/utils/security"
import picturePreview from "@/components/picture-preview.vue"
import { useI18n } from "vue-i18n"

const { t } = useI18n()

const props = defineProps({
    content: {
        type: String,
        required: false,
    },
    mentioned_items: {
        type: Array,
        required: false,
        default: () => [],
    },
    images: {
        type: Array,
        required: false,
        default: () => [],
    },
    attachments: {
        type: Array,
        required: false,
        default: () => [],
    },
    channel: {
        type: String,
        required: false,
        default: "",
    },
})

const channelLabelMap = {
    web: () => t("chat.channelWeb"),
    api: () => t("chat.channelApi"),
    im: () => t("chat.channelIm"),
}

const channelLabel = computed(() => {
    if (!props.channel) return ""
    const label = channelLabelMap[props.channel]
    return typeof label === "function" ? label() : label || props.channel
})

const channelClass = computed(() => (props.channel ? `channel-${props.channel}` : ""))

const containerRef = ref(null)
const hasImages = computed(() => props.images && props.images.length > 0)
const hasAttachments = computed(() => props.attachments && props.attachments.length > 0)

const hydrateImages = async () => {
    await nextTick()
    await hydrateProtectedFileImages(containerRef.value)
}

watch(() => props.images, hydrateImages)
onMounted(hydrateImages)

const reviewImg = ref(false)
const reviewUrl = ref("")

const previewImage = (event) => {
    const src = event.target?.src
    if (src) {
        reviewUrl.value = src
        reviewImg.value = true
    }
}

const getIcon = (type) => {
    if (!type) return "file"
    if (type.startsWith("audio/")) return "sound"
    if (type.startsWith("video/")) return "play-circle"
    if (type.startsWith("application/pdf")) return "file-pdf"
    return "file"
}

const formatBytes = (bytes = 0) => {
    if (!bytes || bytes === 0) return "0 B"
    const k = 1024
    const sizes = ["B", "KB", "MB", "GB", "TB"]
    const i = Math.floor(Math.log(bytes) / Math.log(k))
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + " " + sizes[i]
}

const closePreImg = () => {
    reviewImg.value = false
    reviewUrl.value = ""
}
</script>
<style scoped lang="less">
.user_msg_container {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    gap: 6px;
    width: 100%;
}

.mentioned_items {
    display: flex;
    flex-wrap: wrap;
    gap: 6px;
    justify-content: flex-end;
    max-width: 100%;
    margin-bottom: 2px;
}

.mentioned_tag {
    display: inline-flex;
    align-items: center;
    gap: 4px;
    padding: 3px 8px;
    border-radius: 5px;
    font-size: 12px;
    font-weight: 500;
    max-width: 200px;
    cursor: default;
    transition: all 0.15s;
    background: rgba(7, 192, 95, 0.06);
    border: 1px solid rgba(7, 192, 95, 0.2);
    color: var(--td-text-color-primary);

    &.kb-tag {
        .tag_icon {
            color: var(--td-brand-color);
        }
    }

    &.faq-tag {
        .tag_icon {
            color: var(--td-warning-color);
        }
    }

    &.file-tag {
        .tag_icon {
            color: var(--td-text-color-secondary);
        }
    }

    .tag_icon {
        font-size: 13px;
        display: flex;
        align-items: center;
    }

    .tag_name {
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
        color: currentColor;
    }
}

.user_msg {
    width: max-content;
    max-width: 776px;
    display: flex;
    padding: 10px 12px;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    gap: 4px;
    flex: 1 0 0;
    border-radius: 4px;
    background: #8ce97f;
    margin-left: auto;
    color: #000000e6;
    font-size: 15px;
    text-align: justify;
    word-break: break-all;
    max-width: 100%;
    box-sizing: border-box;
}

.user_images {
    display: flex;
    flex-wrap: wrap;
    gap: 6px;
    justify-content: flex-end;
    max-width: 100%;
}

.user_image_thumb {
    width: 120px;
    height: 120px;
    object-fit: cover;
    border-radius: 6px;
    cursor: pointer;
    border: 1px solid var(--td-border-level-2-color, #e7e7e7);
    transition: opacity 0.2s;

    &:hover {
        opacity: 0.85;
    }
}

.user_attachments {
    display: flex;
    flex-direction: column;
    gap: 8px;
    align-items: flex-end;
    max-width: 100%;
}

.user_attachment_item {
    display: flex;
    align-items: center;
    gap: 10px;
}

.attachment_link {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    padding: 8px 10px;
    border-radius: 6px;
    background: rgba(255, 255, 255, 0.04);
    color: var(--td-text-color-primary);
    text-decoration: none;
    border: 1px solid var(--td-border-level-2-color, #e7e7e7);
}

.attachment_icon {
    font-size: 20px;
}

.attachment_meta {
    display: flex;
    flex-direction: column;
    font-size: 13px;
}

.attachment_name {
    max-width: 360px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

.attachment_size {
    color: var(--td-text-color-placeholder);
    font-size: 12px;
}

.channel_tag {
    display: inline-flex;
    align-items: center;
    padding: 1px 6px;
    border-radius: 3px;
    font-size: 11px;
    font-weight: 500;
    line-height: 18px;
    background: var(--td-bg-color-secondarycontainer);
    color: var(--td-text-color-placeholder);
    border: 1px solid var(--td-border-level-2-color, #e7e7e7);

    &.channel-web {
        color: var(--td-brand-color);
        background: var(--td-brand-color-light);
        border-color: var(--td-brand-color-2, rgba(0, 82, 217, 0.1));
    }

    &.channel-api {
        color: var(--td-success-color);
        background: var(--td-success-color-1, rgba(0, 168, 112, 0.06));
        border-color: var(--td-success-color-2, rgba(0, 168, 112, 0.15));
    }

    &.channel-im {
        color: var(--td-warning-color);
        background: var(--td-warning-color-1, rgba(237, 123, 0, 0.06));
        border-color: var(--td-warning-color-2, rgba(237, 123, 0, 0.15));
    }
}

html[theme-mode="dark"] {
    .user_msg {
        background: var(--td-brand-color-3);
        color: rgba(255, 255, 255, 0.9);
    }
}
</style>
