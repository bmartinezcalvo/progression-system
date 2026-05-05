<script setup>
import { computed, ref, watch } from 'vue';
import { CdxButton, CdxCard, CdxIcon, CdxProgressBar, CdxTextInput } from '@wikimedia/codex';
import {
  cdxIconAdd,
  cdxIconArrowNext,
  cdxIconArrowPrevious,
  cdxIconCheck,
  cdxIconCollapse,
  cdxIconClose
  ,
  cdxIconExpand
} from '@wikimedia/codex-icons';

import ChromeWrapper from './components/ChromeWrapper.vue';

const surveySteps = [ 'welcome', 'reason', 'topics', 'languages', 'email' ];
const currentSurveyStepIndex = ref( 0 );
const currentView = ref( 'survey' );
const resumeSurveyStepIndex = ref( 1 );

const selectedReason = ref( '' );
const selectedTopics = ref( [] );
const selectedLanguages = ref( [ 'English', 'Spanish' ] );
const email = ref( '' );

const userName = 'CaptainBird';
const userInitials = 'Ca';

const reasonOptions = [
  {
    key: 'fix',
    title: 'Fix a typo',
    description: 'To fix a typo or error in a Wikipedia article'
  },
  {
    key: 'share',
    title: 'Share what I know',
    description: 'To add or change information to a Wikipedia article'
  },
  {
    key: 'image',
    title: 'Add an image',
    description: 'To add a photo or image to a Wikipedia article'
  },
  {
    key: 'class-project',
    title: "I'm here for a class or project",
    description: "I'm participating in a program, class, or event"
  },
  {
    key: 'exploring',
    title: 'Just exploring',
    description: 'Not sure yet — I want to see how this works'
  },
  {
    key: 'something-else',
    title: 'Something else',
    description: ''
  }
];

const topicOptions = [
  { key: 'reference', title: 'Reference', description: 'Description' },
  { key: 'culture', title: 'Culture', description: 'Description' },
  { key: 'geography', title: 'Geography', description: 'Description' },
  { key: 'history', title: 'History', description: 'Description' },
  { key: 'human-activities-1', title: 'Human activities', description: 'Description' },
  { key: 'human-activities-2', title: 'Human activities', description: 'Description' },
  { key: 'mathematics', title: 'Mathematics', description: 'Description' },
  { key: 'nature', title: 'Nature', description: 'Description' }
];

const availableLanguages = [ 'English', 'Spanish', 'French', 'Catalan', 'German' ];
const otherReason = ref( '' );

const progressionTasks = ref( [
  {
    key: 'welcome-survey',
    title: 'Complete Welcome Survey',
    description: 'Respond four quick questions to personalize your experience',
    duration: '1 min',
    buttonLabel: 'Complete survey',
    action: 'survey',
    completed: false
  },
  {
    key: 'wiki-minute',
    title: 'Watch a Wiki Minute video',
    description: '60-second intro to how Wikipedia is written, sourced, and edited',
    duration: '1 min',
    buttonLabel: 'Watch video',
    action: 'video',
    completed: false
  },
  {
    key: 'five-pillars',
    title: 'Five Pillars quiz',
    description: 'Review the core principles that guide editing on Wikipedia',
    duration: '15 min',
    buttonLabel: 'Review pillars',
    action: 'quiz',
    completed: false
  },
  {
    key: 'first-edit',
    title: 'First suggested edit',
    description: 'A small and easy edit picked for you',
    duration: '5 min',
    buttonLabel: 'Start edit',
    action: 'edit',
    completed: false
  },
  {
    key: 'enrich-article',
    title: 'Enrich an article',
    description: 'Choose one way to improve an article: citation, image, or link',
    duration: '15 min',
    buttonLabel: 'Enrich article',
    action: 'enrich',
    completed: false
  }
] );

const editsCount = ref( 0 );
const thanksCount = ref( 0 );
const streakCount = ref( 0 );
const isProgressionExpanded = ref( false );

const currentSurveyStep = computed( () => surveySteps[ currentSurveyStepIndex.value ] );
const completedTaskCount = computed(
  () => progressionTasks.value.filter( ( task ) => task.completed ).length
);
const currentTaskIndex = computed(
  () => progressionTasks.value.findIndex( ( task ) => !task.completed )
);
const currentTaskNumber = computed( () => currentTaskIndex.value + 1 );
const shouldCollapseProgression = computed( () => completedTaskCount.value < 2 );
const visibleTasks = computed( () => progressionTasks.value );

const surveyProgressValue = computed( () => {
  switch ( currentSurveyStep.value ) {
    case 'welcome':
      return 10;
    case 'reason':
      return 25;
    case 'topics':
      return 50;
    case 'languages':
      return 80;
    case 'email':
      return 100;
    default:
      return 0;
  }
} );

const canAdvanceSurvey = computed( () => {
  switch ( currentSurveyStep.value ) {
    case 'reason':
      return Boolean( selectedReason.value );
    case 'topics':
      return selectedTopics.value.length >= 2;
    case 'languages':
      return selectedLanguages.value.length > 0;
    case 'email':
      return email.value.trim().length > 0;
    default:
      return false;
  }
} );

watch(
  currentView,
  ( nextView ) => {
    if ( nextView === 'home' ) {
      isProgressionExpanded.value = !shouldCollapseProgression.value;
    }
  },
  { immediate: true }
);

function goToNextSurveyStep() {
  if ( currentSurveyStepIndex.value < surveySteps.length - 1 ) {
    currentSurveyStepIndex.value += 1;
    resumeSurveyStepIndex.value = currentSurveyStepIndex.value;
  }
}

function goToPreviousSurveyStep() {
  if ( currentSurveyStepIndex.value > 1 ) {
    currentSurveyStepIndex.value -= 1;
    resumeSurveyStepIndex.value = currentSurveyStepIndex.value;
  }
}

function toggleTopic( key ) {
  if ( selectedTopics.value.includes( key ) ) {
    selectedTopics.value = selectedTopics.value.filter( ( item ) => item !== key );
    return;
  }

  selectedTopics.value = [ ...selectedTopics.value, key ];
}

function removeLanguage( language ) {
  selectedLanguages.value = selectedLanguages.value.filter( ( item ) => item !== language );
}

function addLanguage() {
  const nextLanguage = availableLanguages.find(
    ( language ) => !selectedLanguages.value.includes( language )
  );

  if ( nextLanguage ) {
    selectedLanguages.value = [ ...selectedLanguages.value, nextLanguage ];
  }
}

function skipSurvey() {
  resumeSurveyStepIndex.value = Math.max( currentSurveyStepIndex.value, 1 );
  currentView.value = 'home';
}

function completeSurvey() {
  progressionTasks.value[ 0 ].completed = true;
  resumeSurveyStepIndex.value = 1;
  currentView.value = 'home';
}

function completeCurrentTask() {
  const index = currentTaskIndex.value;

  if ( index === -1 ) {
    return;
  }

  const task = progressionTasks.value[ index ];

  if ( task.action === 'survey' ) {
    currentSurveyStepIndex.value = resumeSurveyStepIndex.value;
    currentView.value = 'survey';
    return;
  }

  task.completed = true;

  if ( task.action === 'edit' ) {
    editsCount.value += 1;
    streakCount.value = Math.max( streakCount.value, 1 );
  }

  if ( task.action === 'enrich' ) {
    editsCount.value += 4;
    thanksCount.value += 2;
    streakCount.value = Math.max( streakCount.value, 3 );
  }

  if ( currentTaskIndex.value > 1 ) {
    isProgressionExpanded.value = true;
  }
}

function toggleProgressionExpanded() {
  if ( shouldCollapseProgression.value ) {
    isProgressionExpanded.value = !isProgressionExpanded.value;
  }
}

function goToHomepage() {
  currentView.value = 'home';
}
</script>

<template>
  <ChromeWrapper @go-home="goToHomepage">
    <section v-if="currentView === 'survey'" class="survey-page">
      <div v-if="currentSurveyStep === 'welcome'" class="survey-page__panel survey-page__panel--welcome">
        <div class="survey-page__copy">
          <h1 class="survey-page__title">
            Welcome, {{ userName }}
          </h1>
          <p class="survey-page__lead">
            Wikipedia is built by people like you.
          </p>
          <p class="survey-page__body">
            Respond four quick questions to customize your experience.
          </p>
        </div>

        <div class="survey-page__welcome-actions">
          <CdxButton
            class="survey-page__full-button"
            action="progressive"
            weight="primary"
            size="large"
            @click="goToNextSurveyStep"
          >
            Customize experience
          </CdxButton>
          <CdxButton
            class="survey-page__full-button"
            size="large"
            @click="skipSurvey"
          >
            Skip
          </CdxButton>
        </div>
      </div>

      <div v-else class="survey-page__panel survey-page__panel--step">
        <div class="survey-page__meta">
          <div class="survey-page__skip-row">
            <button class="survey-page__skip-link" type="button" @click="skipSurvey">
              Skip
            </button>
          </div>

          <div class="survey-page__progress">
            <CdxProgressBar
              aria-label="Survey progress"
              :value="surveyProgressValue"
              :max="100"
            />
          </div>
        </div>

        <template v-if="currentSurveyStep === 'reason'">
          <div class="survey-page__copy survey-page__copy--step">
            <h1 class="survey-page__title survey-page__title--question">
              Why did you create your account?
            </h1>
            <p class="survey-page__body">
              Please select one response.
            </p>
          </div>

          <div class="survey-page__options">
            <button
              v-for="option in reasonOptions"
              :key="option.key"
              class="survey-card"
              :class="{ 'survey-card--selected': selectedReason === option.key }"
              type="button"
              @click="selectedReason = option.key"
            >
              <span class="survey-card__title">{{ option.title }}</span>
              <span v-if="option.description" class="survey-card__description">{{ option.description }}</span>
              <CdxTextInput
                v-if="option.key === 'something-else'"
                v-model="otherReason"
                class="survey-card__input"
                placeholder="Add your reason"
                aria-label="Add your reason"
                @click.stop
              />
            </button>
          </div>
        </template>

        <template v-else-if="currentSurveyStep === 'topics'">
          <div class="survey-page__copy survey-page__copy--step">
            <h1 class="survey-page__title survey-page__title--question">
              What topics you are interested in?
            </h1>
            <p class="survey-page__body">
              Select at least 2 topics.
            </p>
          </div>

          <div class="survey-page__options">
            <button
              v-for="option in topicOptions"
              :key="option.key"
              class="survey-card"
              :class="{ 'survey-card--selected': selectedTopics.includes( option.key ) }"
              type="button"
              @click="toggleTopic( option.key )"
            >
              <span class="survey-card__title">{{ option.title }}</span>
              <span class="survey-card__description">{{ option.description }}</span>
            </button>
          </div>
        </template>

        <template v-else-if="currentSurveyStep === 'languages'">
          <div class="survey-page__copy survey-page__copy--step">
            <h1 class="survey-page__title survey-page__title--question">
              Which languages do you read and write in?
            </h1>
            <p class="survey-page__body">
              Wikipedia is available in nearly 300 languages. Please, select all the languages
              you read and write in.
            </p>
          </div>

          <div class="survey-page__chips">
            <button
              v-for="language in selectedLanguages"
              :key="language"
              class="survey-chip"
              type="button"
              @click="removeLanguage( language )"
            >
              <span>{{ language }}</span>
              <span class="survey-chip__icon">
                <CdxIcon :icon="cdxIconClose" />
              </span>
            </button>

            <button class="survey-chip survey-chip--add" type="button" @click="addLanguage">
              <span class="survey-chip__icon">
                <CdxIcon :icon="cdxIconAdd" />
              </span>
              <span>Add</span>
            </button>
          </div>
        </template>

        <template v-else-if="currentSurveyStep === 'email'">
          <div class="survey-page__copy survey-page__copy--step">
            <h1 class="survey-page__title survey-page__title--question">
              Almost completed
            </h1>
            <p class="survey-page__body">
              We noticed you didn’t enter an email when creating this account.
            </p>
            <p class="survey-page__body survey-page__body--small">
              Your email is just needed for account recovery if you ever lose your password.
              Your email address is not revealed when other users contact you.
            </p>
          </div>

          <label class="survey-page__field">
            <span class="survey-page__field-label">Email address (optional)</span>
            <CdxTextInput
              v-model="email"
              input-type="email"
              placeholder="Entre your email"
              aria-label="Email address optional"
            />
          </label>
        </template>

        <div class="survey-page__footer-nav">
          <CdxButton
            v-if="currentSurveyStep !== 'reason'"
            class="survey-page__nav-link"
            weight="quiet"
            @click="goToPreviousSurveyStep"
          >
            <template #icon>
              <CdxIcon :icon="cdxIconArrowPrevious" />
            </template>
            Previous
          </CdxButton>

          <CdxButton
            v-if="currentSurveyStep !== 'email' ? canAdvanceSurvey : true"
            class="survey-page__nav-link survey-page__nav-link--next"
            weight="quiet"
            action="progressive"
            @click="currentSurveyStep === 'email' ? completeSurvey() : goToNextSurveyStep()"
          >
            {{ currentSurveyStep === 'email' ? 'Complete' : 'Next' }}
            <template #icon>
              <CdxIcon :icon="cdxIconArrowNext" />
            </template>
          </CdxButton>
        </div>
      </div>
    </section>

    <section v-else class="homepage">
      <header class="homepage__profile">
        <button class="homepage__avatar" type="button" aria-label="Add profile photo">
          {{ userInitials }}
        </button>

        <div class="homepage__profile-copy">
          <h1 class="homepage__name">
            {{ userName }}
          </h1>
          <p class="homepage__meta">
            Joined today • Beginner editor
          </p>
        </div>
      </header>

      <nav class="homepage__tabs" aria-label="User navigation">
        <a class="homepage__tab homepage__tab--active" href="#">Homepage</a>
        <a class="homepage__tab" href="#">User page</a>
        <a class="homepage__tab" href="#">Talk</a>
      </nav>

      <section class="module module--path">
        <div
          class="progression-list"
          :class="{ 'progression-list--collapsed': shouldCollapseProgression && !isProgressionExpanded }"
        >
          <article
            v-for="( task, index ) in visibleTasks"
            :key="task.key"
            class="progression-item"
            :class="{
              'progression-item--complete': task.completed,
              'progression-item--active': index === currentTaskIndex
            }"
          >
            <div class="progression-item__rail">
              <div class="progression-item__marker">
                <CdxIcon v-if="task.completed" :icon="cdxIconCheck" />
                <span v-else>{{ index + 1 }}</span>
              </div>
              <div
                v-if="index < progressionTasks.length - 1"
                class="progression-item__line"
              ></div>
            </div>

            <div class="progression-item__content">
              <div class="progression-item__heading">
                <h2 class="progression-item__title">
                  {{ task.title }}
                </h2>
                <span class="progression-item__duration">{{ task.duration }}</span>
              </div>

              <p class="progression-item__description">
                {{ task.description }}
              </p>

              <p v-if="task.completed" class="progression-item__status">
                <CdxIcon :icon="cdxIconCheck" />
                <span>Completed</span>
              </p>

              <CdxButton
                v-if="index === currentTaskIndex"
                class="progression-item__button"
                action="progressive"
                weight="primary"
                @click="completeCurrentTask"
              >
                {{ task.buttonLabel }}
                <template #icon>
                  <CdxIcon :icon="cdxIconArrowNext" />
                </template>
              </CdxButton>
            </div>
          </article>
        </div>

        <div v-if="shouldCollapseProgression" class="homepage__see-all-wrap">
          <div class="homepage__see-all-gradient" aria-hidden="true"></div>
          <button
            class="homepage__see-all"
            type="button"
            @click="toggleProgressionExpanded"
          >
            <span>{{ isProgressionExpanded ? 'Fewer steps' : 'More steps' }}</span>
            <CdxIcon :icon="isProgressionExpanded ? cdxIconCollapse : cdxIconExpand" />
          </button>
        </div>
      </section>

      <section class="module module--mentor">
        <div class="module__mentor">
          <img
            class="module__mentor-avatar"
            src="https://images.unsplash.com/photo-1500375592092-40eb2168fd21?auto=format&fit=crop&w=240&q=80"
            alt="Mentor avatar"
          >
          <div class="module__mentor-copy">
            <h2 class="module__title module__title--small">
              Raydann is your mentor
            </h2>
            <p class="module__body">
              You’ve been assigned an experienced editor to help you with editing
            </p>
            <CdxButton class="module__mentor-button">
              Ask your mentor
            </CdxButton>
          </div>
        </div>
      </section>

      <section class="homepage__impact-grid">
        <article class="impact-card">
          <strong class="impact-card__value">{{ editsCount }}/5</strong>
          <span class="impact-card__label">edits</span>
        </article>
        <article class="impact-card">
          <strong class="impact-card__value">{{ thanksCount }}</strong>
          <span class="impact-card__label">thanks</span>
        </article>
        <article class="impact-card">
          <strong class="impact-card__value">{{ streakCount }}</strong>
          <span class="impact-card__label">streak days</span>
        </article>
      </section>

      <CdxCard class="module-card-link" url="#">
        <template #title>
          Get help with editing
        </template>
        <template #description>
          Ask the help desk or read help pages
        </template>
      </CdxCard>
    </section>
  </ChromeWrapper>
</template>

<style scoped>
.survey-page,
.homepage {
  min-height: 100%;
  background-color: var(--background-color-base);
}

.survey-page__panel,
.homepage {
  max-width: 30rem;
  padding: var(--spacing-150);
}

.survey-page__panel--welcome,
.survey-page__panel--step {
  display: flex;
  flex-direction: column;
}

.survey-page__panel--welcome {
  gap: var(--spacing-300);
}

.survey-page__panel--step {
  gap: var(--spacing-200);
}

.survey-page__copy {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-50);
}

.survey-page__title,
.homepage__name {
  margin: 0;
  color: var(--color-emphasized);
  font-family: var(--font-family-heading-main);
  font-size: 1.75rem;
  line-height: 2.375rem;
  font-weight: var(--font-weight-regular);
}

.survey-page__title--question {
  font-family: var(--font-family-system-sans);
  font-weight: var(--font-weight-bold);
}

.survey-page__lead,
.survey-page__body,
.homepage__meta,
.module__body,
.progression-item__description {
  margin: 0;
  color: var(--color-emphasized);
  font-size: 1rem;
  line-height: 1.5;
}

.survey-page__lead {
  font-size: 1.125rem;
}

.survey-page__body--small,
.impact-card__label,
.progression-item__duration {
  color: var(--color-subtle);
  font-size: 0.875rem;
  line-height: 1.45;
}

.survey-page__progress :deep(.cdx-progress-bar) {
  width: 100%;
}

.survey-page__welcome-actions {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-75);
  margin-bottom: 80px;
}

.survey-page__full-button {
  width: 100%;
  justify-content: center;
}

.survey-page__meta {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-75);
}

.survey-page__skip-row {
  display: flex;
  justify-content: flex-end;
}

.survey-page__skip-link,
.homepage__see-all {
  border: 0;
  padding: 0;
  background: transparent;
  font-size: 1rem;
  font-weight: var(--font-weight-bold);
}

.survey-page__skip-link {
  color: var(--color-emphasized);
}

.survey-page__options {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-50);
}

.survey-card {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-50);
  width: 100%;
  border: var(--border-base);
  background-color: var(--background-color-base);
  padding: 12px;
  text-align: left;
}

.survey-card--selected {
  border-color: var(--border-color-progressive);
  background-color: var(--background-color-progressive-subtle);
}

.progression-item__title,
.module__title,
.homepage__tab {
  color: var(--color-emphasized);
  font-size: 1rem;
  font-weight: var(--font-weight-bold);
  line-height: 1.4;
}

.survey-card__title {
  color: var(--color-emphasized);
  font-size: 1.125rem;
  font-weight: var(--font-weight-bold);
  line-height: 1.75rem;
}

.survey-card__description {
  color: var(--color-subtle);
  font-size: 1rem;
  line-height: 1.5;
}

.survey-card__input {
  margin-top: var(--spacing-50);
}

.survey-page__chips {
  display: flex;
  flex-wrap: wrap;
  gap: var(--spacing-75);
}

.survey-chip {
  display: inline-flex;
  align-items: center;
  gap: var(--spacing-50);
  border: 1px solid var(--border-color-progressive);
  border-radius: 999px;
  background-color: var(--background-color-progressive-subtle);
  padding: 0.625rem 0.875rem;
  color: var(--color-progressive);
  font-size: 1rem;
  font-weight: var(--font-weight-bold);
}

.survey-chip--add {
  border-color: var(--border-color-interactive);
  background-color: var(--background-color-base);
  color: var(--color-base);
}

.survey-chip__icon,
.survey-page__nav-link,
.homepage__see-all,
.progression-item__status {
  display: inline-flex;
  align-items: center;
  gap: var(--spacing-50);
}

.survey-page__field {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-50);
}

.survey-page__field-label {
  color: var(--color-emphasized);
  font-size: 1rem;
  line-height: 1.4;
}

.survey-page__footer-nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  position: sticky;
  bottom: 0;
  z-index: 1;
  margin-top: auto;
  padding-top: 12px;
  padding-bottom: calc(12px + env(safe-area-inset-bottom, 0px));
  background:
    linear-gradient(
      180deg,
      rgb(255 255 255 / 0) 0%,
      rgb(255 255 255 / 0.96) 24%,
      rgb(255 255 255 / 1) 100%
    );
}

.survey-page__nav-link--next {
  margin-inline-start: auto;
}

.survey-page__nav-link--next :deep(.cdx-icon) {
  color: var(--color-progressive);
}

.homepage {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-150);
}

.homepage__profile {
  display: flex;
  align-items: center;
  gap: var(--spacing-100);
}

.homepage__avatar {
  width: 3.5rem;
  height: 3.5rem;
  border: 1px solid var(--border-color-subtle);
  border-radius: 50%;
  background: #edf3ff;
  color: var(--color-emphasized);
  font-size: 1.5rem;
  font-weight: var(--font-weight-regular);
}

.homepage__profile-copy {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-25);
}

.homepage__tabs {
  display: flex;
  gap: var(--spacing-125);
  border-bottom: 1px solid var(--border-color-subtle);
}

.homepage__tab {
  text-decoration: none;
}

.homepage__tab--active {
  text-decoration: underline;
  text-underline-offset: 0.3rem;
}

.module {
  border: var(--border-base);
  border-color: var(--border-color-subtle);
  border-radius: var(--border-radius-base);
  padding: 12px;
  background-color: var(--background-color-base);
}

.module--path {
  border: 0;
  padding: 0;
}

.progression-list {
  display: flex;
  flex-direction: column;
  gap: 0;
}

.progression-list--collapsed {
  max-height: 23.5rem;
  overflow: hidden;
}

.progression-item {
  display: grid;
  grid-template-columns: 2rem 1fr;
  gap: var(--spacing-75);
}

.progression-item__rail {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.progression-item__marker {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border: 1px solid #c8ccd1;
  border-radius: 50%;
  background-color: var(--background-color-base);
  color: var(--color-subtle);
  font-size: 14px;
  font-weight: var(--font-weight-bold);
  line-height: 1;
}

.progression-item__line {
  width: 1.5px;
  flex: 1 1 auto;
  min-height: 2.75rem;
  background-color: var(--border-color-muted);
}

.progression-item--complete .progression-item__marker {
  border-color: var(--color-icon-success);
  background-color: var(--color-icon-success);
  color: var(--color-inverted);
}

.progression-item--complete .progression-item__line {
  background-color: var(--border-color-success);
}

.progression-item--active .progression-item__marker {
  border: 3px solid var(--background-color-progressive-subtle);
  background-color: var(--background-color-progressive);
  color: var(--color-inverted);
}

.progression-item__content {
  display: flex;
  flex-direction: column;
  gap: 4px;
  padding-bottom: 24px;
}

.progression-item__heading {
  display: flex;
  align-items: center;
  gap: var(--spacing-50);
  min-height: 24px;
}

.progression-item__status {
  color: var(--color-icon-success);
  font-size: 14px;
  line-height: 22px;
}

.progression-item__button {
  align-self: flex-start;
}

.homepage__see-all {
  align-self: center;
  position: relative;
  z-index: 1;
  margin-bottom: 8px;
  color: var(--color-emphasized);
  pointer-events: auto;
}

.homepage__see-all-wrap {
  position: absolute;
  right: 0;
  bottom: 0;
  left: 0;
  display: flex;
  justify-content: center;
  align-items: flex-end;
  height: 104px;
  pointer-events: none;
}

.homepage__see-all-gradient {
  position: absolute;
  inset: 0;
  background: linear-gradient(180deg, rgb(255 255 255 / 0) 0%, rgb(255 255 255 / 0.95) 58%, rgb(255 255 255 / 1) 100%);
}

.module__mentor {
  display: flex;
  gap: var(--spacing-100);
}

.module__mentor-avatar {
  display: block;
  width: 40px;
  height: 40px;
  min-width: 40px;
  min-height: 40px;
  flex-shrink: 0;
  border: 1px solid var(--border-color-subtle);
  border-radius: 50%;
  object-fit: cover;
  object-position: center;
  aspect-ratio: 1 / 1;
}

.module__mentor-copy {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-50);
}

.module__title--small {
  margin: 0;
}

.module__mentor-button {
  align-self: flex-start;
}

.homepage__impact-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: var(--spacing-75);
}

.impact-card {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-25);
  border: var(--border-base);
  padding: 12px;
  background-color: var(--background-color-base);
}

.impact-card__value {
  color: var(--color-emphasized);
  font-size: 16px;
  line-height: 1;
}

.survey-page__panel--step .survey-page__copy {
  gap: var(--spacing-50);
}

.survey-page__panel--step .survey-page__options,
.survey-page__panel--step .survey-page__chips,
.survey-page__panel--step .survey-page__field {
  margin-top: 0;
}

.survey-page__field :deep(.cdx-text-input__input) {
  min-height: 44px;
  padding-right: 12px;
  padding-left: 12px;
}

.survey-page__title--question {
  font-size: 20px;
  line-height: 30px;
}

.progression-item__marker :deep(.cdx-icon) {
  width: 12px;
  height: 12px;
}

.progression-item__title {
  font-size: 16px;
  line-height: 22px;
}

.progression-item__description,
.progression-item__duration {
  font-size: 14px;
  line-height: 22px;
}

.module--help {
  background-color: var(--background-color-neutral-subtle);
}

.module--help .module__title {
  margin-top: 0;
  margin-bottom: 4px;
}

.progression-item__title {
  margin: 0;
  padding-top: 0;
  padding-bottom: 0;
}

.module-card-link:deep(.cdx-card) {
  border-color: var(--border-color-interactive);
  border-radius: var(--border-radius-base);
}

@media (min-width: 640px) {
  .survey-page__panel,
  .homepage {
    max-width: 33rem;
    padding: var(--spacing-150) var(--spacing-100);
  }

  .survey-page__title,
  .homepage__name {
    font-size: 1.625rem;
    line-height: 2.25rem;
  }

}
</style>
