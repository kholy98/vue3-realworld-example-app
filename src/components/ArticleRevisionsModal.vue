<template>
  <div v-if="isOpen" class="modal-overlay">
    <div class="modal-backdrop" @click.self="close"></div>
    <div class="modal-container">
      <div class="modal-header">
        <h3>Article Revisions</h3>
        <button class="close-button" @click="close">×</button>
      </div>
      <div class="modal-content">
        <div v-if="loading" class="loading-state">
          <i class="ion-load-c"></i> Loading revisions...
        </div>
        <div v-else-if="error" class="error-state">
          {{ error }}
        </div>
        <div v-else-if="revisions.length === 0" class="empty-state">
          No revisions found
        </div>
        <div v-else class="revisions-list">
          <div
            v-for="revision in revisions"
            :key="revision.id"
            class="revision-item"
          >
            <div class="revision-header">
              <span class="revision-date">
                {{ formatDate(revision.createdAt) }}
              </span>
              <span class="revision-author" v-if="revision.user">
                by {{ revision.user.username }}
              </span>
            </div>
            <div class="revision-content">
              <h4>{{ revision.title }}</h4>
              <p class="revision-description">{{ revision.description }}</p>
            </div>
            <div class="revision-actions">
              <button
                class="btn btn-sm btn-info"
                @click="viewRevision(revision)"
              >
                View
              </button>
              <button
                class="btn btn-sm btn-warning"
                :disabled="reverting"
                @click="revertToRevision(revision)"
              >
                {{ reverting ? 'Reverting...' : 'Revert to this version' }}
              </button>
            </div>
          </div>
        </div>
      </div>
      <div class="modal-footer">
        <button class="btn btn-secondary" @click="close">Close</button>
      </div>
    </div>
  </div>

  <!-- Revision Detail Modal -->
  <div v-if="showRevisionDetail" class="modal-overlay">
    <div class="modal-backdrop" @click.self="showRevisionDetail = false"></div>
    <div class="modal-container wide-modal">
      <div class="modal-header">
        <h3>Revision Details</h3>
        <button class="close-button" @click="showRevisionDetail = false">×</button>
      </div>
      <div class="modal-content">
        <div v-if="selectedRevision">
          <div class="form-group">
            <label class="form-label">Title</label>
            <input class="form-input" :value="selectedRevision.title" readonly>
          </div>
          <div class="form-group">
            <label class="form-label">Description</label>
            <textarea class="form-textarea" :value="selectedRevision.description" readonly rows="2"></textarea>
          </div>
          <div class="form-group">
            <label class="form-label">Body</label>
            <textarea class="form-textarea" :value="selectedRevision.body" readonly rows="10"></textarea>
          </div>
        </div>
      </div>
      <div class="modal-footer">
        <button class="btn btn-secondary" @click="showRevisionDetail = false">Close</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'
import { api } from 'src/services'
import type { ArticleRevision } from 'src/services/api'

interface Props {
  articleSlug: string
  isOpen: boolean
}

const props = defineProps<Props>()
const emit = defineEmits<{
  (e: 'update:isOpen', value: boolean): void
  (e: 'reverted', article: any): void
}>()

const revisions = ref<ArticleRevision[]>([])
const loading = ref(false)
const error = ref('')
const reverting = ref(false)
const selectedRevision = ref<ArticleRevision | null>(null)
const showRevisionDetail = ref(false)

const close = () => {
  console.log('Closing modal...');
  emit('update:isOpen', false);
}

const formatDate = (dateString: string) => {
  return new Date(dateString).toLocaleString()
}

const loadRevisions = async () => {
  if (!props.isOpen) return

  loading.value = true
  error.value = ''
  try {
    const response = await api.articles.getArticleRevisions(props.articleSlug)
    revisions.value = response.data.revisions || []
    console.log('Revisions loaded:', revisions.value);
  } catch (err: any) {
    error.value = err.response?.data?.errors?.body?.[0] || 'Failed to load revisions'
    console.error('Error loading revisions:', error.value);
  } finally {
    loading.value = false
  }
}

const viewRevision = (revision: ArticleRevision) => {
  console.log('Viewing revision:', revision);
  selectedRevision.value = revision
  showRevisionDetail.value = true
}

const revertToRevision = async (revision: ArticleRevision) => {
  if (!confirm('Are you sure you want to revert to this version?')) return

  reverting.value = true
  try {
    const response = await api.articles.revertArticleRevision(
      props.articleSlug,
      revision.id
    )
    emit('reverted', response.data.article)
    close()
  } catch (err: any) {
    error.value = err.response?.data?.errors?.body?.[0] || 'Failed to revert revision'
  } finally {
    reverting.value = false
  }
}

// Load revisions when modal opens
watch(() => props.isOpen, (isOpen) => {
  console.log('Modal open state:', isOpen);
  if (isOpen) {
    loadRevisions();
  }
});
</script>

<style scoped>
/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999; /* Increased z-index */
}

.modal-backdrop {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 9998; /* Backdrop z-index */
}

.modal-container {
  background: white;
  border-radius: 8px;
  width: 90%;
  max-width: 600px;
  max-height: 80vh;
  display: flex;
  flex-direction: column;
  z-index: 10000; /* Highest z-index */
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.3);
}

.wide-modal {
  max-width: 800px;
}

.modal-header {
  padding: 1rem;
  border-bottom: 1px solid #e5e5e5;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.modal-header h3 {
  margin: 0;
  font-size: 1.25rem;
  font-weight: 600;
}

.close-button {
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  padding: 0;
  width: 30px;
  height: 30px;
  line-height: 1;
}

.close-button:hover {
  color: #ff4757;
}

.modal-content {
  padding: 1rem;
  overflow-y: auto;
  flex: 1;
}

.modal-footer {
  padding: 1rem;
  border-top: 1px solid #e5e5e5;
  display: flex;
  justify-content: flex-end;
}

/* Form Styles */
.form-group {
  margin-bottom: 1rem;
}

.form-label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
  color: #333;
}

.form-input,
.form-textarea {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
  background-color: #f8f9fa;
}

.form-textarea {
  resize: vertical;
  min-height: 60px;
}

/* Button Styles */
.btn {
  padding: 0.5rem 1rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  background: white;
  cursor: pointer;
  font-size: 0.875rem;
  transition: all 0.2s;
}

.btn:hover {
  background-color: #f8f9fa;
}

.btn-sm {
  padding: 0.25rem 0.5rem;
  font-size: 0.75rem;
}

.btn-info {
  background-color: #17a2b8;
  color: white;
  border-color: #17a2b8;
}

.btn-info:hover {
  background-color: #138496;
  border-color: #117a8b;
}

.btn-warning {
  background-color: #ffc107;
  color: #212529;
  border-color: #ffc107;
}

.btn-warning:hover {
  background-color: #e0a800;
  border-color: #d39e00;
}

.btn-secondary {
  background-color: #6c757d;
  color: white;
  border-color: #6c757d;
}

.btn-secondary:hover {
  background-color: #5a6268;
  border-color: #545b62;
}

.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* Revisions List */
.revisions-list {
  max-height: 400px;
  overflow-y: auto;
}

.revision-item {
  border: 1px solid #e9ecef;
  border-radius: 6px;
  padding: 1rem;
  margin-bottom: 0.75rem;
  background-color: #f8f9fa;
}

.revision-item:hover {
  background-color: #e9ecef;
}

.revision-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 0.5rem;
  font-size: 0.875rem;
  color: #6c757d;
}

.revision-content h4 {
  margin: 0 0 0.5rem 0;
  font-size: 1rem;
  color: #212529;
  font-weight: 600;
}

.revision-description {
  margin: 0;
  color: #6c757d;
  font-size: 0.875rem;
  line-height: 1.4;
}

.revision-actions {
  margin-top: 0.75rem;
  display: flex;
  gap: 0.5rem;
}

/* State Styles */
.loading-state,
.empty-state {
  text-align: center;
  padding: 2rem;
  color: #6c757d;
  font-style: italic;
}

.error-state {
  background-color: #f8d7da;
  color: #721c24;
  padding: 1rem;
  border-radius: 4px;
  border: 1px solid #f5c6cb;
}
</style>