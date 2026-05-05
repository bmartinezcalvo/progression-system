<script setup>
import { computed, onBeforeUnmount, onMounted, ref, watch } from 'vue';
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
import encyclopediaPillarImage from './assets/five-pillars/encyclopedia.svg';
import neutralPillarImage from './assets/five-pillars/neutral.svg';
import freeContentPillarImage from './assets/five-pillars/free-content.svg';
import civilityPillarImage from './assets/five-pillars/civility.svg';
import noRulesPillarImage from './assets/five-pillars/no-rules.svg';

const surveySteps = [ 'welcome', 'reason', 'topics', 'languages', 'email' ];
const currentSurveyStepIndex = ref( 0 );
const currentView = ref( 'survey' );
const resumeSurveyStepIndex = ref( 1 );
const currentQuizIndex = ref( 0 );
const currentSuggestedEditIndex = ref( 0 );
const suggestedEditOrdinal = ref( 1 );
const recentlyCompletedTaskKey = ref( '' );
const showArticlePathEntry = ref( false );
let articleScrollTimeout = null;

const selectedReason = ref( '' );
const selectedTopics = ref( [] );
const selectedLanguages = ref( [ 'English', 'Spanish' ] );
const email = ref( '' );
const selectedQuizAnswers = ref( {} );

const userName = 'CaptainBird';
const userInitials = 'Ca';
const wikiMinuteWelcomeImage =
  'https://commons.wikimedia.org/wiki/Special:Redirect/file/A%20Wiki%20Minute%20-%20extract%20021.jpg';
const articleTitle = 'Paris';
const articleSections = [
  {
    title: 'History',
    body: 'Paris developed from a Celtic settlement on the Île de la Cité into a major political, cultural, and commercial center of Europe. Over centuries the city expanded on both banks of the Seine and became closely associated with art, science, and revolution.'
  },
  {
    title: 'Geography',
    body: 'The city lies in north-central France on a broad bend of the Seine River. Paris is organized into twenty arrondissements and is surrounded by dense suburbs that form one of the largest metropolitan regions in Europe.'
  },
  {
    title: 'Culture',
    body: 'Paris is known for its museums, architecture, fashion, literature, and cuisine. Landmarks such as the Eiffel Tower, the Louvre, and Notre-Dame make it one of the most visited cities in the world.'
  }
];

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

const quizSteps = [
  {
    key: 'encyclopedia',
    image: encyclopediaPillarImage,
    imageAlt: 'Illustration for the encyclopedia pillar',
    title: 'Wikipedia is an encyclopedia',
    description:
      'Wikipedia is not a place for opinions, ads, social networking, random information, or personal content. It’s also not a dictionary, news site, or instruction manual.',
    example: '"Michael Jordan is my favorite basketball player"',
    question: 'Does the previous sentence belong in a Wikipedia article?',
    errorMessage:
      'Not quite. This is an opinion — not everyone agrees, and it can’t be verified with a source.',
    correctKey: 'b',
    options: [
      {
        key: 'a',
        label: 'A',
        text: 'Yes — it\'s a well-known fact about Michael Jordan.'
      },
      {
        key: 'b',
        label: 'B',
        text: 'No — it\'s an opinion. Wikipedia needs verifiable facts, not judgments.'
      },
      {
        key: 'c',
        label: 'C',
        text: 'Yes, if enough people agree with it.'
      }
    ]
  },
  {
    key: 'neutral',
    image: neutralPillarImage,
    imageAlt: 'Illustration for the neutral point of view pillar',
    title: 'Wikipedia is written from a neutral point of view',
    description:
      'Wikipedia articles should be neutral and based on reliable sources. They explain different points of view fairly, without promoting any. Personal opinions or experiences are not included.',
    example: '"Rosalía is the greatest singer of all time."',
    question: 'Does the previous sentence belong in a Wikipedia article?',
    errorMessage:
      'Not quite. This is an opinion and it can’t be verified with a source.',
    correctKey: 'a',
    options: [
      {
        key: 'a',
        label: 'A',
        text: 'No — it\'s an opinion. Wikipedia needs verifiable facts, not judgments.'
      },
      {
        key: 'b',
        label: 'B',
        text: 'Yes — it\'s a well-known fact about Rosalía.'
      },
      {
        key: 'c',
        label: 'C',
        text: 'Yes, if enough people agree with it.'
      }
    ]
  },
  {
    key: 'free-content',
    image: freeContentPillarImage,
    imageAlt: 'Illustration for the free content pillar',
    title: 'Wikipedia is free content that anyone can use, edit, and distribute',
    description:
      'Anyone can use, edit, and redistribute Wikipedia’s content. The content you add must be freely licensed or in the public domain. That means you can’t copy text from other sources.',
    example:
      'You want to improve an article about a painting. You find a perfect paragraph in an art history book and copy it word for word into the article.',
    question: 'Is this the right approach?',
    errorMessage:
      'Copyright still applies. You need to paraphrase and write in your own words.',
    correctKey: 'b',
    options: [
      {
        key: 'a',
        label: 'A',
        text: 'Yes — if it\'s accurate and well-written, it improves the article.'
      },
      {
        key: 'b',
        label: 'B',
        text: 'No — copying copyrighted text violates Wikipedia\'s free content policy.'
      },
      {
        key: 'c',
        label: 'C',
        text: 'Only if you credit the book in the references.'
      }
    ]
  },
  {
    key: 'civility',
    image: civilityPillarImage,
    imageAlt: 'Illustration for the civility pillar',
    title: 'Wikipedia editors come from everywhere.',
    description:
      'Wikipedia editors come from everywhere. Disagreements happen — but they should be resolved through discussion, not personal attacks. Assume other editors mean well.',
    example:
      'Another editor removes a sentence you added with no explanation in the edit summary.',
    question: 'What\'s the best response?',
    errorMessage:
      'Reverting without discussion or assuming bad faith creates edit wars.',
    correctKey: 'b',
    options: [
      {
        key: 'a',
        label: 'A',
        text: 'Revert their edit immediately — your version was better.'
      },
      {
        key: 'b',
        label: 'B',
        text: 'Leave a message on their talk page asking why they removed it.'
      },
      {
        key: 'c',
        label: 'C',
        text: 'Report them to an admin for vandalism.'
      }
    ]
  },
  {
    key: 'no-rules',
    image: noRulesPillarImage,
    imageAlt: 'Illustration for the no firm rules pillar',
    title: 'Wikipedia has no firm rules',
    description:
      'Wikipedia\'s policies exist to serve the goal of building a free encyclopedia — not the other way around. Use good judgment. If a rule prevents a good outcome, it may not apply.',
    example:
      'A new editor adds accurate, well-sourced content but formats it slightly wrong. A veteran editor removes the entire contribution because it doesn\'t follow the style guide.',
    question: 'Did the veteran editor act in the spirit of Wikipedia?',
    errorMessage:
      'Not quite. This is an opinion and it can’t be verified with a source.',
    correctKey: 'b',
    options: [
      {
        key: 'a',
        label: 'A',
        text: 'Yes — rules exist to be followed consistently.'
      },
      {
        key: 'b',
        label: 'B',
        text: 'No — good content shouldn\'t be removed over formatting. Fix the format, keep the content.'
      },
      {
        key: 'c',
        label: 'C',
        text: 'It depends on how wrong the formatting was.'
      }
    ]
  }
];

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
    buttonLabel: 'See suggested edits',
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
const suggestedEdits = ref( [
  {
    key: 'isabella-duplicate-link',
    articleTitle: 'Isabella I of Castile',
    articleDescription: 'Queen of Castile and León',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/thumb/8/8c/Isabella_I_of_Castile.jpg/320px-Isabella_I_of_Castile.jpg',
    articleSnippet:
      'Isabella and Ferdinand are known for being the first monarchs to be referred to as the queen and king of Spain, respectively. Their actions included completion of the Reconquista, the Alhambra Decree which ordered the mass expulsion of Jews from Spain, initiating the Spanish Inquisition, financing Christopher Columbus\'s 1492 voyage to the New World, and establishing the Spanish Empire.',
    taskTitle: 'Remove duplicate link',
    taskDescription:
      'This link appears more than once in this section. Help readers navigate more easily by removing repeated links.',
    question: 'Remove this extra link from the paragraph?',
    successTitle: 'Removed duplicate link',
    successMessage: 'Thank you for improving Wikipedia.',
    primaryLabel: 'Remove link',
    secondaryLabel: 'Dismiss'
  },
  {
    key: 'isabella-uk-spelling',
    articleTitle: 'Isabella I of Castile',
    articleDescription: 'Queen of Castile and León',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/thumb/8/8c/Isabella_I_of_Castile.jpg/320px-Isabella_I_of_Castile.jpg',
    articleSnippet:
      'Girón. Desiring to depose Henry and establish Infante Alfonso on the throne, Pacheco and his followers circulated rumors that Infanta Joanna was actually the child of Beltrán de la Cueva and demanded that Alfonso be named the King\'s successor.',
    taskTitle: 'Change English spelling',
    taskDescription:
      'This word uses a different variety of English than the one used in the rest of this article.',
    question: 'Replace “rumors” with “rumours”?',
    successTitle: 'Updated the spelling',
    successMessage: 'Thanks for keeping the article consistent.',
    primaryLabel: 'Apply change',
    secondaryLabel: 'Dismiss'
  },
  {
    key: 'paris-date-link',
    articleTitle: 'Paris',
    articleDescription: 'Capital and largest city of France',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/thumb/a/a8/Tour_Eiffel_Wikimedia_Commons.jpg/320px-Tour_Eiffel_Wikimedia_Commons.jpg',
    articleSnippet:
      'Paris developed from a Celtic settlement on the Île de la Cité into a major political, cultural, and commercial center of Europe. Over centuries the city expanded on both banks of the Seine.',
    taskTitle: 'Link a key date',
    taskDescription:
      'Add a helpful link so readers can quickly learn more about this historical period.',
    question: 'Add a link to “Île de la Cité”?',
    successTitle: 'Linked a key topic',
    successMessage: 'That link will help readers explore the topic.',
    primaryLabel: 'Add link',
    secondaryLabel: 'Dismiss'
  }
] );

const currentSurveyStep = computed( () => surveySteps[ currentSurveyStepIndex.value ] );
const currentQuizStep = computed( () => quizSteps[ currentQuizIndex.value ] );
const completedTaskCount = computed(
  () => progressionTasks.value.filter( ( task ) => task.completed ).length
);
const currentTaskIndex = computed(
  () => progressionTasks.value.findIndex( ( task ) => !task.completed )
);
const currentTaskNumber = computed( () => currentTaskIndex.value + 1 );
const activeProgressTask = computed(
  () => ( currentTaskIndex.value >= 0 ? progressionTasks.value[ currentTaskIndex.value ] : null )
);
const shouldCollapseProgression = computed( () => completedTaskCount.value < 2 );
const visibleTasks = computed( () => {
  if ( shouldCollapseProgression.value && !isProgressionExpanded.value ) {
    return progressionTasks.value.slice( 0, 2 );
  }

  return progressionTasks.value;
} );
const currentQuizAnswer = computed(
  () => selectedQuizAnswers.value[ currentQuizStep.value.key ] || ''
);
const currentQuizAnswerIsCorrect = computed(
  () => currentQuizAnswer.value === currentQuizStep.value.correctKey
);
const remainingSuggestedEdits = computed(
  () => suggestedEdits.value.filter( ( suggestion ) => !suggestion.resolved )
);
const currentSuggestedEdit = computed(
  () => remainingSuggestedEdits.value[ currentSuggestedEditIndex.value ] || null
);
const currentSurveyQuestionNumber = computed( () => {
  switch ( currentSurveyStep.value ) {
    case 'reason':
      return 1;
    case 'topics':
      return 2;
    case 'languages':
      return 3;
    case 'email':
      return 4;
    default:
      return 0;
  }
} );
const shouldShowChromeFooter = computed(
  () => ![ 'survey', 'quiz', 'suggested-edits' ].includes( currentView.value )
);

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

    if ( nextView !== 'article' ) {
      showArticlePathEntry.value = false;
      if ( articleScrollTimeout ) {
        clearTimeout( articleScrollTimeout );
        articleScrollTimeout = null;
      }
    }
  },
  { immediate: true }
);

onMounted( () => {
  window.addEventListener( 'scroll', handleArticleScroll, { passive: true } );
} );

onBeforeUnmount( () => {
  window.removeEventListener( 'scroll', handleArticleScroll );
  if ( articleScrollTimeout ) {
    clearTimeout( articleScrollTimeout );
  }
} );

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

  if ( task.action === 'quiz' ) {
    currentQuizIndex.value = 0;
    currentView.value = 'quiz';
    return;
  }

  if ( task.action === 'edit' ) {
    goToSuggestedEdits();
    return;
  }

  task.completed = true;
  recentlyCompletedTaskKey.value = task.key;
  setTimeout( () => {
    if ( recentlyCompletedTaskKey.value === task.key ) {
      recentlyCompletedTaskKey.value = '';
    }
  }, 550 );
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

function goToArticle() {
  currentView.value = 'article';
}

function goToSuggestedEdits() {
  currentSuggestedEditIndex.value = 0;
  suggestedEditOrdinal.value = 1;
  currentView.value = 'suggested-edits';
}

function resolveSuggestedEdit( mode, suggestionKey ) {
  const suggestion = currentSuggestedEdit.value;

  if ( !suggestion || suggestion.key !== suggestionKey ) {
    return;
  }

  suggestion.resolving = mode;

  if ( mode === 'complete' && !progressionTasks.value.find( ( task ) => task.key === 'first-edit' )?.completed ) {
    progressionTasks.value = progressionTasks.value.map( ( task ) =>
      task.key === 'first-edit' ? { ...task, completed: true } : task
    );
    recentlyCompletedTaskKey.value = 'first-edit';
    editsCount.value += 1;
    streakCount.value = Math.max( streakCount.value, 1 );
    setTimeout( () => {
      if ( recentlyCompletedTaskKey.value === 'first-edit' ) {
        recentlyCompletedTaskKey.value = '';
      }
    }, 550 );
  }

  setTimeout( () => {
    suggestion.resolved = true;
    suggestion.resolving = '';
    suggestedEditOrdinal.value += 1;

    const remaining = remainingSuggestedEdits.value;
    if ( remaining.length === 0 ) {
      currentView.value = 'home';
      currentSuggestedEditIndex.value = 0;
      return;
    }

    currentSuggestedEditIndex.value = Math.min(
      currentSuggestedEditIndex.value,
      remaining.length - 1
    );
  }, mode === 'complete' ? 1700 : 420 );
}

function handleArticleScroll() {
  if ( currentView.value !== 'article' || !activeProgressTask.value ) {
    return;
  }

  showArticlePathEntry.value = false;

  if ( articleScrollTimeout ) {
    clearTimeout( articleScrollTimeout );
  }

  articleScrollTimeout = setTimeout( () => {
    if ( currentView.value === 'article' && activeProgressTask.value ) {
      showArticlePathEntry.value = true;
    }
  }, 2000 );
}

function selectQuizAnswer( optionKey ) {
  selectedQuizAnswers.value = {
    ...selectedQuizAnswers.value,
    [ currentQuizStep.value.key ]: optionKey
  };
}

function goToPreviousQuizStep() {
  if ( currentQuizIndex.value > 0 ) {
    currentQuizIndex.value -= 1;
    return;
  }

  currentView.value = 'home';
}

function goToNextQuizStep() {
  if ( !currentQuizAnswer.value ) {
    return;
  }

  if ( currentQuizIndex.value < quizSteps.length - 1 ) {
    currentQuizIndex.value += 1;
    return;
  }

  progressionTasks.value = progressionTasks.value.map( ( task ) =>
    task.key === 'five-pillars' ? { ...task, completed: true } : task
  );
  recentlyCompletedTaskKey.value = 'five-pillars';
  setTimeout( () => {
    if ( recentlyCompletedTaskKey.value === 'five-pillars' ) {
      recentlyCompletedTaskKey.value = '';
    }
  }, 550 );
  currentView.value = 'home';
}

function getQuizOptionState( optionKey ) {
  if ( !currentQuizAnswer.value ) {
    return '';
  }

  if ( optionKey === currentQuizStep.value.correctKey ) {
    return 'success';
  }

  if ( optionKey === currentQuizAnswer.value && !currentQuizAnswerIsCorrect.value ) {
    return 'error';
  }

  return '';
}
</script>

<template>
  <ChromeWrapper
    :show-footer="shouldShowChromeFooter"
    @go-home="goToHomepage"
    @go-article="goToArticle"
  >
    <section v-if="currentView === 'survey'" class="survey-page">
      <div v-if="currentSurveyStep === 'welcome'" class="survey-page__panel survey-page__panel--welcome">
        <img
          class="survey-page__hero-image"
          :src="wikiMinuteWelcomeImage"
          alt="Illustration of Wikipedia contributors around a laptop"
        >

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
            <CdxButton class="survey-page__skip-link" weight="quiet" @click="skipSurvey">
              Skip
            </CdxButton>
          </div>

          <div class="survey-page__progress">
            <div class="survey-page__progress-bar-wrap">
              <CdxProgressBar
                aria-label="Survey progress"
                :value="surveyProgressValue"
                :max="100"
              />
            </div>
            <p class="survey-page__progress-text">
              <strong>{{ currentSurveyQuestionNumber }}</strong> of 4
            </p>
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
            <CdxIcon :icon="cdxIconArrowPrevious" />
            Previous
          </CdxButton>

          <CdxButton
            v-if="canAdvanceSurvey"
            class="survey-page__nav-link survey-page__nav-link--next"
            weight="quiet"
            action="progressive"
            @click="currentSurveyStep === 'email' ? completeSurvey() : goToNextSurveyStep()"
          >
            {{ currentSurveyStep === 'email' ? 'Complete' : 'Next' }}
            <CdxIcon :icon="cdxIconArrowNext" />
          </CdxButton>
        </div>
      </div>
    </section>

    <section v-else-if="currentView === 'quiz'" class="quiz-page">
      <header class="quiz-page__header">
        <CdxButton
          class="quiz-page__back"
          weight="quiet"
          aria-label="Go back"
          @click="goToPreviousQuizStep"
        >
          <CdxIcon :icon="cdxIconArrowPrevious" />
        </CdxButton>
        <h1 class="quiz-page__header-title">
          Five pillars quiz
        </h1>
      </header>

      <div class="quiz-page__progress-block">
        <CdxProgressBar
          aria-label="Quiz progress"
          :value="currentQuizIndex + 1"
          :max="quizSteps.length"
        />
        <p class="quiz-page__progress-text">
          <strong>{{ currentQuizIndex + 1 }}</strong> of {{ quizSteps.length }}
        </p>
      </div>

      <article class="quiz-page__panel">
        <img
          class="quiz-page__thumbnail"
          :src="currentQuizStep.image"
          :alt="currentQuizStep.imageAlt"
        >

        <div class="quiz-page__copy">
          <h2 class="quiz-page__title">
            {{ currentQuizStep.title }}
          </h2>
          <p class="quiz-page__description">
            {{ currentQuizStep.description }}
          </p>
        </div>

        <section class="quiz-example">
          <p class="quiz-example__label">
            Example:
          </p>
          <p class="quiz-example__text">
            {{ currentQuizStep.example }}
          </p>
        </section>

        <section class="quiz-question">
          <h3 class="quiz-question__title">
            {{ currentQuizStep.question }}
          </h3>

          <div class="quiz-question__options">
            <article
              v-for="option in currentQuizStep.options"
              :key="option.key"
              class="quiz-option"
              :class="{
                'quiz-option--success': getQuizOptionState( option.key ) === 'success',
                'quiz-option--error': getQuizOptionState( option.key ) === 'error'
              }"
            >
              <button
                class="quiz-option__button"
                type="button"
                @click="selectQuizAnswer( option.key )"
              >
                <span class="quiz-option__key">{{ option.label }}</span>
                <span class="quiz-option__text">{{ option.text }}</span>
              </button>

              <p
                v-if="getQuizOptionState( option.key ) === 'error'"
                class="quiz-option__message quiz-option__message--error"
              >
                {{ currentQuizStep.errorMessage }}
              </p>
            </article>
          </div>
        </section>
      </article>

      <div class="quiz-page__footer-nav">
        <CdxButton
          v-if="currentQuizIndex > 0"
          class="quiz-page__nav-link"
          weight="quiet"
          @click="goToPreviousQuizStep"
        >
          <template #icon>
            <CdxIcon :icon="cdxIconArrowPrevious" />
          </template>
          Previous
        </CdxButton>

        <CdxButton
          v-if="currentQuizAnswer"
          class="quiz-page__nav-link quiz-page__nav-link--next"
          weight="quiet"
          action="progressive"
          @click="goToNextQuizStep"
        >
          {{ currentQuizIndex === quizSteps.length - 1 ? 'Complete quiz' : 'Next' }}
          <template #icon>
            <CdxIcon :icon="cdxIconArrowNext" />
          </template>
        </CdxButton>
      </div>
    </section>

    <section v-else-if="currentView === 'suggested-edits'" class="suggested-edits-page">
      <header class="suggested-edits-page__header">
        <CdxButton
          class="suggested-edits-page__back"
          weight="quiet"
          aria-label="Go back"
          @click="goToHomepage"
        >
          <CdxIcon :icon="cdxIconArrowPrevious" />
        </CdxButton>
        <h1 class="suggested-edits-page__header-title">
          Suggested edits
        </h1>
      </header>

      <p class="suggested-edits-page__count">
        <strong>{{ suggestedEditOrdinal }}</strong> of 32 suggestions
      </p>

      <div class="suggested-edits-page__viewport">
        <div
          class="suggested-edits-page__track"
          :style="{ transform: `translateX(calc(${ currentSuggestedEditIndex * -88 }% - ${ currentSuggestedEditIndex * 16 }px))` }"
        >
          <article
            v-for="suggestion in remainingSuggestedEdits"
            :key="suggestion.key"
            class="suggested-edit-card"
            :class="{
              'suggested-edit-card--active': suggestion.key === currentSuggestedEdit?.key,
              'suggested-edit-card--completing': suggestion.resolving === 'complete',
              'suggested-edit-card--dismissing': suggestion.resolving === 'dismiss'
            }"
          >
            <div class="suggested-edit-card__body">
              <img
                class="suggested-edit-card__thumbnail"
                :src="suggestion.articleImage"
                :alt="suggestion.articleTitle"
              >
              <div class="suggested-edit-card__copy">
                <h2 class="suggested-edit-card__title">
                  {{ suggestion.articleTitle }}
                </h2>
                <p class="suggested-edit-card__description">
                  {{ suggestion.articleDescription }}
                </p>
              </div>

              <div class="suggested-edit-card__excerpt">
                {{ suggestion.articleSnippet }}
              </div>
            </div>

            <div
              v-if="suggestion.resolving === 'complete'"
              class="suggested-edit-card__panel suggested-edit-card__panel--success"
            >
              <div class="suggested-edit-card__success-row">
                <span class="suggested-edit-card__success-icon">
                  <CdxIcon :icon="cdxIconCheck" />
                </span>
                <strong>{{ suggestion.successTitle }}</strong>
              </div>
              <p class="suggested-edit-card__panel-description">
                {{ suggestion.successMessage }}
              </p>
            </div>

            <div
              v-else
              class="suggested-edit-card__panel suggested-edit-card__panel--task"
            >
              <h3 class="suggested-edit-card__panel-title">
                {{ suggestion.taskTitle }}
              </h3>
              <p class="suggested-edit-card__panel-description">
                {{ suggestion.taskDescription }}
              </p>
              <p class="suggested-edit-card__question">
                {{ suggestion.question }}
              </p>

              <div class="suggested-edit-card__actions">
                <CdxButton @click="resolveSuggestedEdit( 'complete', suggestion.key )">
                  {{ suggestion.primaryLabel }}
                </CdxButton>
                <CdxButton @click="resolveSuggestedEdit( 'dismiss', suggestion.key )">
                  {{ suggestion.secondaryLabel }}
                </CdxButton>
                <button class="suggested-edit-card__menu" type="button" aria-label="More actions">
                  ...
                </button>
              </div>
            </div>
          </article>
        </div>
      </div>
    </section>

    <section v-else-if="currentView === 'article'" class="article-page">
      <header class="article-page__header">
        <div class="article-page__title-row">
          <div>
            <h1 class="article-page__title">
              {{ articleTitle }}
            </h1>
            <p class="article-page__subtitle">
              Capital and most populous city of France
            </p>
          </div>
          <a class="article-page__language-link" href="#">84 languages</a>
        </div>

        <nav class="article-page__tabs" aria-label="Article actions">
          <a class="article-page__tab article-page__tab--active" href="#">Article</a>
          <a class="article-page__tab" href="#">Talk</a>
          <a class="article-page__tab" href="#">Read</a>
          <a class="article-page__tab" href="#">Edit</a>
          <a class="article-page__tab" href="#">View history</a>
        </nav>
      </header>

      <div class="article-page__layout">
        <aside class="article-page__infobox">
          <figure class="article-page__media">
            <img
              class="article-page__image"
              src="https://upload.wikimedia.org/wikipedia/commons/e/e6/Paris_Night.jpg"
              alt="Paris skyline"
            >
            <figcaption class="article-page__caption">
              Paris skyline seen from the Seine
            </figcaption>
          </figure>

          <dl class="article-page__facts">
            <div class="article-page__fact">
              <dt>Country</dt>
              <dd>France</dd>
            </div>
            <div class="article-page__fact">
              <dt>Region</dt>
              <dd>Île-de-France</dd>
            </div>
            <div class="article-page__fact">
              <dt>Population</dt>
              <dd>2,102,650</dd>
            </div>
            <div class="article-page__fact">
              <dt>Demonym</dt>
              <dd>Parisian</dd>
            </div>
          </dl>
        </aside>

        <article class="article-page__content">
          <p class="article-page__lead">
            <strong>Paris</strong> is the capital and most populous city of France. Located on the Seine River,
            it has long been a major center of politics, commerce, science, and the arts.
          </p>
          <p class="article-page__lead">
            The city is globally recognized for its architecture, museums, and role in European history.
            It is one of the world’s leading tourist destinations and an important transport hub.
          </p>

          <nav class="article-page__toc" aria-label="Contents">
            <h2 class="article-page__section-label">
              Contents
            </h2>
            <ol class="article-page__toc-list">
              <li v-for="section in articleSections" :key="section.title">
                <a href="#">{{ section.title }}</a>
              </li>
            </ol>
          </nav>

          <section
            v-for="section in articleSections"
            :key="section.title"
            class="article-page__section"
          >
            <h2 class="article-page__section-title">
              {{ section.title }}
            </h2>
            <p class="article-page__paragraph">
              {{ section.body }}
            </p>
          </section>
        </article>
      </div>

      <transition name="article-path-entry">
        <button
          v-if="showArticlePathEntry && activeProgressTask"
          class="article-path-entry"
          type="button"
          @click="goToHomepage"
        >
          <span class="article-path-entry__marker">{{ currentTaskNumber }}</span>
          <span class="article-path-entry__copy">
            <span class="article-path-entry__eyebrow">Continue your path</span>
            <span class="article-path-entry__meta">
              {{ currentTaskNumber }} of {{ progressionTasks.length }} steps • {{ activeProgressTask.title }}
            </span>
          </span>
          <span class="article-path-entry__chevron" aria-hidden="true">
            <CdxIcon :icon="cdxIconArrowNext" />
          </span>
        </button>
      </transition>
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
              'progression-item--active': index === currentTaskIndex,
              'progression-item--recently-completed': recentlyCompletedTaskKey === task.key
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

        <div v-if="shouldCollapseProgression && progressionTasks.length > 2" class="homepage__see-all-wrap">
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
.homepage,
.quiz-page,
.suggested-edits-page,
.article-page {
  min-height: 100%;
  background-color: var(--background-color-base);
}

.survey-page {
  background-color: var(--background-color-neutral-subtle);
}

.survey-page__panel,
.homepage,
.quiz-page,
.suggested-edits-page,
.article-page {
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

.survey-page__hero-image {
  display: block;
  width: 100%;
  height: auto;
  border: 1px solid var(--border-color-subtle);
  border-radius: var(--border-radius-base);
  background-color: var(--background-color-base);
  object-fit: cover;
}

.survey-page__panel--welcome .survey-page__copy {
  margin-top: -28px;
}

.survey-page__panel--step {
  gap: 24px;
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

.survey-page__progress {
  display: flex;
  align-items: center;
  gap: 8px;
}

.survey-page__progress-bar-wrap {
  flex: 1 1 auto;
}

.survey-page__progress-bar-wrap :deep(.cdx-progress-bar) {
  width: 100%;
}

.survey-page__progress-bar-wrap :deep(.cdx-progress-bar__bar),
.survey-page__progress-bar-wrap :deep(.cdx-progress-bar__fill),
.quiz-page__progress-block :deep(.cdx-progress-bar__bar),
.quiz-page__progress-block :deep(.cdx-progress-bar__fill) {
  height: 8px;
}

.survey-page__progress-bar-wrap :deep(.cdx-progress-bar__bar),
.survey-page__progress-bar-wrap :deep(.cdx-progress-bar__fill) {
  height: 8px;
}

.survey-page__progress-text {
  margin: 0;
  color: var(--color-subtle);
  font-size: 14px;
  line-height: 22px;
}

.survey-page__progress-text strong {
  color: var(--color-emphasized);
  font-weight: var(--font-weight-bold);
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
  gap: 24px;
}

.survey-page__skip-row {
  display: flex;
  justify-content: flex-end;
}

.survey-page__skip-link {
  align-self: flex-end;
}

.survey-page__skip-link:deep(.cdx-button),
.homepage__see-all {
  font-size: 1rem;
  font-weight: var(--font-weight-bold);
}

.homepage__see-all {
  border: 0;
  padding: 0;
  background: transparent;
}

.survey-page__options {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.survey-card {
  display: flex;
  flex-direction: column;
  gap: 4px;
  width: 100%;
  border: var(--border-base);
  border-radius: var(--border-radius-base);
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
  gap: 12px;
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

.survey-chip__icon {
  width: 16px;
  height: 16px;
  justify-content: center;
}

.survey-chip__icon :deep(.cdx-icon) {
  width: 16px;
  height: 16px;
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
  background-color: var(--background-color-neutral-subtle);
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
  gap: 12px;
}

.homepage__avatar {
  width: 48px;
  height: 48px;
  border: 1px solid var(--border-color-subtle);
  border-radius: 50%;
  background: #edf3ff;
  color: var(--color-emphasized);
  font-size: 16px;
  line-height: 22px;
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
  color: var(--color-subtle);
  font-size: 14px;
  line-height: 22px;
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
  position: relative;
}

.progression-list--collapsed {
  overflow: visible;
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
  position: relative;
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
  transition:
    background-color 180ms ease,
    border-color 180ms ease,
    color 180ms ease,
    box-shadow 180ms ease,
    transform 180ms ease;
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
  background-color: var(--background-color-progressive);
  color: var(--color-inverted);
  box-shadow: 0 0 0 3px var(--background-color-progressive-subtle);
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
  padding-top: 0;
  padding-bottom: 0;
}

.progression-item__button {
  align-self: flex-start;
}

.homepage__see-all {
  align-self: center;
  position: relative;
  z-index: 1;
  margin-bottom: 0;
  color: var(--color-emphasized);
  pointer-events: auto;
}

.homepage__see-all-wrap {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: flex-end;
  height: 80px;
  margin-top: -64px;
  pointer-events: none;
}

.homepage__see-all-gradient {
  position: absolute;
  right: 0;
  bottom: 0;
  left: 0;
  height: 80px;
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
  gap: 0;
}

.module__title--small {
  margin: 0;
}

.module__mentor-copy .module__body {
  margin-top: 4px;
}

.module__mentor-button {
  align-self: flex-start;
  margin-top: 8px;
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
  border-color: var(--border-color-muted);
  border-radius: var(--border-radius-base);
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

.quiz-page {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.quiz-page__header {
  display: flex;
  align-items: center;
  gap: 4px;
  padding-bottom: 12px;
  border-bottom: 1px solid var(--border-color-subtle);
}

.quiz-page__back {
  flex-shrink: 0;
}

.quiz-page__header-title {
  margin: 0;
  color: var(--color-emphasized);
  font-family: var(--font-family-system-sans);
  font-size: 20px;
  line-height: 30px;
  font-weight: var(--font-weight-bold);
}

.quiz-page__progress-block {
  display: grid;
  grid-template-columns: minmax(0, 1fr) auto;
  align-items: center;
  gap: 8px;
}

.quiz-page__progress-text {
  margin: 0;
  color: var(--color-subtle);
  font-size: 14px;
  line-height: 22px;
}

.quiz-page__progress-text strong {
  color: var(--color-emphasized);
  font-weight: var(--font-weight-bold);
}

.quiz-page__progress-block :deep(.cdx-progress-bar) {
  width: 100%;
}

.quiz-page__panel {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.quiz-page__thumbnail {
  width: 40px;
  height: 40px;
  border: 1px solid var(--border-color-subtle);
  border-radius: var(--border-radius-base);
  object-fit: cover;
}

.quiz-page__copy {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.quiz-page__title {
  margin: 0;
  padding-bottom: 8px;
  border-bottom: 1px solid var(--border-color-subtle);
  color: var(--color-emphasized);
  font-family: var(--font-family-heading-main);
  font-size: 20px;
  line-height: 30px;
  font-weight: var(--font-weight-regular);
}

.quiz-page__description {
  margin: 0;
  color: var(--color-emphasized);
  font-size: 16px;
  line-height: 32px;
}

.quiz-example {
  display: flex;
  flex-direction: column;
  gap: 4px;
  border: var(--border-base);
  border-color: var(--border-color-muted);
  border-radius: var(--border-radius-base);
  background-color: var(--background-color-neutral-subtle);
  padding: 12px;
}

.quiz-example__label {
  margin: 0;
  color: var(--color-base);
  font-size: 14px;
  line-height: 22px;
}

.quiz-example__text {
  margin: 0;
  color: var(--color-emphasized);
  font-family: var(--font-family-heading-main);
  font-size: 16px;
  line-height: 22px;
}

.quiz-question {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.quiz-question__title {
  margin: 0;
  color: var(--color-emphasized);
  font-size: 18px;
  line-height: 28px;
  font-weight: var(--font-weight-bold);
}

.quiz-question__options {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.quiz-option {
  border: var(--border-base);
  border-color: var(--border-color-base);
  border-radius: var(--border-radius-base);
  background-color: var(--background-color-base);
}

.quiz-option--success {
  border-color: var(--border-color-success);
  background-color: var(--background-color-success-subtle);
}

.quiz-option--error {
  border-color: var(--border-color-error);
  background-color: var(--background-color-error-subtle);
}

.quiz-option__button {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  width: 100%;
  border: 0;
  background: transparent;
  padding: 12px;
  text-align: left;
}

.quiz-option__key {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 24px;
  height: 24px;
  flex-shrink: 0;
  border: 1px solid var(--border-color-base);
  border-radius: 50%;
  color: var(--color-subtle);
  font-size: 14px;
  line-height: 1;
}

.quiz-option--success .quiz-option__key {
  border-color: var(--color-icon-success);
  background-color: var(--color-icon-success);
  color: var(--color-inverted);
}

.quiz-option--error .quiz-option__key {
  border-color: var(--color-error);
  background-color: var(--color-error);
  color: var(--color-inverted);
}

.quiz-option__text {
  color: var(--color-emphasized);
  font-size: 16px;
  line-height: 22px;
}

.quiz-option__message {
  margin: 0;
  padding: 0 12px 12px 12px;
  font-size: 14px;
  line-height: 22px;
}

.quiz-option__message--error {
  color: var(--color-error);
}

.quiz-page__footer-nav {
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

.quiz-page__nav-link {
  display: inline-flex;
  align-items: center;
  gap: var(--spacing-50);
}

.quiz-page__nav-link--next {
  margin-inline-start: auto;
}

.quiz-page__nav-link--next :deep(.cdx-icon) {
  color: var(--color-progressive);
}

.suggested-edits-page {
  display: flex;
  flex-direction: column;
  gap: 16px;
  min-height: 100%;
  background-color: var(--background-color-neutral-subtle);
}

.suggested-edits-page__header {
  display: flex;
  align-items: center;
  gap: 4px;
  padding-bottom: 12px;
  border-bottom: 1px solid var(--border-color-subtle);
}

.suggested-edits-page__header-title {
  margin: 0;
  color: var(--color-emphasized);
  font-family: var(--font-family-system-sans);
  font-size: 20px;
  line-height: 30px;
  font-weight: var(--font-weight-bold);
}

.suggested-edits-page__count {
  margin: 0;
  color: var(--color-subtle);
  font-size: 14px;
  line-height: 20px;
  text-align: center;
}

.suggested-edits-page__count strong {
  color: var(--color-emphasized);
  font-weight: var(--font-weight-bold);
}

.suggested-edits-page__viewport {
  overflow: hidden;
  margin-inline: -12px;
  padding-inline: 12px;
}

.suggested-edits-page__track {
  display: flex;
  gap: 16px;
  transition: transform 280ms ease;
}

.suggested-edit-card {
  width: 88%;
  flex: 0 0 88%;
  border: 1px solid var(--border-color-subtle);
  border-radius: var(--border-radius-base);
  background-color: var(--background-color-base);
  overflow: hidden;
  opacity: 0.72;
  transition:
    transform 260ms ease,
    opacity 260ms ease;
}

.suggested-edit-card--active {
  opacity: 1;
}

.suggested-edit-card--completing {
  transform: scale(0.985);
}

.suggested-edit-card--dismissing {
  opacity: 0;
  transform: translateX(-20px) scale(0.98);
}

.suggested-edit-card__body {
  padding: 12px;
}

.suggested-edit-card__thumbnail {
  display: block;
  width: 62px;
  height: 62px;
  margin-bottom: 12px;
  border: 1px solid var(--border-color-subtle);
  border-radius: var(--border-radius-base);
  object-fit: cover;
}

.suggested-edit-card__copy {
  display: flex;
  flex-direction: column;
  gap: 4px;
  padding-bottom: 12px;
  border-bottom: 1px solid var(--border-color-subtle);
}

.suggested-edit-card__title {
  margin: 0;
  color: var(--color-emphasized);
  font-family: var(--font-family-heading-main);
  font-size: 20px;
  line-height: 30px;
  font-weight: var(--font-weight-regular);
}

.suggested-edit-card__description {
  margin: 0;
  color: var(--color-subtle);
  font-size: 16px;
  line-height: 22px;
}

.suggested-edit-card__excerpt {
  padding-top: 12px;
  color: var(--color-base);
  font-size: 16px;
  line-height: 22px;
}

.suggested-edit-card__panel {
  padding: 12px;
}

.suggested-edit-card__panel--task {
  background-color: var(--background-color-neutral);
}

.suggested-edit-card__panel--success {
  background-color: var(--background-color-success-subtle);
}

.suggested-edit-card__panel-title,
.suggested-edit-card__question {
  margin: 0;
  color: var(--color-emphasized);
  font-size: 16px;
  line-height: 22px;
  font-weight: var(--font-weight-bold);
}

.suggested-edit-card__panel-description {
  margin: 8px 0 0 0;
  color: var(--color-base);
  font-size: 16px;
  line-height: 22px;
}

.suggested-edit-card__question {
  margin-top: 16px;
}

.suggested-edit-card__actions {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
  margin-top: 12px;
}

.suggested-edit-card__menu {
  border: 0;
  background: transparent;
  color: var(--color-subtle);
  font-size: 24px;
  line-height: 1;
}

.suggested-edit-card__success-row {
  display: flex;
  align-items: center;
  gap: 8px;
  color: var(--color-emphasized);
}

.suggested-edit-card__success-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  border-radius: 50%;
  background-color: var(--color-icon-success);
  color: var(--color-inverted);
}

.progression-item__marker :deep(.cdx-icon) {
  width: 12px;
  height: 12px;
  color: var(--color-inverted);
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
  background-color: var(--background-color-neutral-subtle) !important;
}

.module-card-link:deep(.cdx-card__text__title),
.module-card-link:deep(.cdx-card__text__description) {
  background-color: transparent;
}

.progression-item__status :deep(.cdx-icon) {
  color: var(--color-success);
}

.progression-item--recently-completed .progression-item__marker {
  animation: progression-marker-complete 420ms ease;
}

@keyframes progression-marker-complete {
  0% {
    transform: scale(1);
  }

  40% {
    transform: scale(1.14);
  }

  100% {
    transform: scale(1);
  }
}

.article-page {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.article-page__header {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.article-page__title-row {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.article-page__title {
  margin: 0;
  color: var(--color-emphasized);
  font-family: var(--font-family-heading-main);
  font-size: 32px;
  line-height: 1.2;
  font-weight: var(--font-weight-regular);
}

.article-page__subtitle {
  margin: 0;
  color: var(--color-subtle);
  font-size: 14px;
  line-height: 22px;
}

.article-page__language-link,
.article-page__toc-list a {
  color: var(--color-progressive);
  text-decoration: none;
}

.article-page__tabs {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  padding-bottom: 8px;
  border-bottom: 1px solid var(--border-color-subtle);
}

.article-page__tab {
  color: var(--color-emphasized);
  font-size: 14px;
  line-height: 22px;
  text-decoration: none;
}

.article-page__tab--active {
  font-weight: var(--font-weight-bold);
  text-decoration: underline;
  text-underline-offset: 0.3rem;
}

.article-page__layout {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.article-page__infobox {
  order: 2;
  border: 1px solid var(--border-color-subtle);
  background-color: var(--background-color-neutral-subtle);
}

.article-page__media {
  margin: 0;
}

.article-page__image {
  display: block;
  width: 100%;
  height: auto;
}

.article-page__caption {
  padding: 8px 12px;
  color: var(--color-subtle);
  font-size: 12px;
  line-height: 18px;
}

.article-page__facts {
  margin: 0;
}

.article-page__fact {
  display: grid;
  grid-template-columns: 6rem 1fr;
  gap: 8px;
  padding: 8px 12px;
  border-top: 1px solid var(--border-color-subtle);
}

.article-page__fact dt {
  color: var(--color-subtle);
  font-size: 14px;
  line-height: 22px;
}

.article-page__fact dd {
  margin: 0;
  color: var(--color-emphasized);
  font-size: 14px;
  line-height: 22px;
}

.article-page__content {
  order: 1;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.article-page__lead,
.article-page__paragraph {
  margin: 0;
  color: var(--color-emphasized);
  font-size: 16px;
  line-height: 28px;
}

.article-page__toc {
  border: 1px solid var(--border-color-subtle);
  background-color: var(--background-color-neutral-subtle);
  padding: 12px;
}

.article-page__section-label {
  margin: 0 0 8px 0;
  color: var(--color-emphasized);
  font-size: 16px;
  line-height: 22px;
  font-weight: var(--font-weight-bold);
}

.article-page__toc-list {
  margin: 0;
  padding-left: 18px;
}

.article-page__toc-list li + li {
  margin-top: 4px;
}

.article-page__section {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.article-page__section-title {
  margin: 0;
  padding-bottom: 8px;
  border-bottom: 1px solid var(--border-color-subtle);
  color: var(--color-emphasized);
  font-family: var(--font-family-heading-main);
  font-size: 24px;
  line-height: 32px;
  font-weight: var(--font-weight-regular);
}

.article-path-entry {
  position: fixed;
  right: 16px;
  bottom: calc(16px + env(safe-area-inset-bottom, 0px));
  left: 16px;
  z-index: 20;
  display: flex;
  align-items: flex-start;
  gap: 16px;
  border: 1px solid var(--border-color-muted);
  border-radius: var(--border-radius-base);
  background-color: var(--background-color-neutral-subtle);
  box-shadow: 0 6px 16px rgb(0 0 0 / 0.12);
  padding: 12px;
  text-align: left;
}

.article-path-entry__marker {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 24px;
  height: 24px;
  flex-shrink: 0;
  margin-top: -1px;
  border-radius: 50%;
  background-color: var(--background-color-progressive);
  color: var(--color-inverted);
  font-size: 14px;
  line-height: 1;
}

.article-path-entry__copy {
  display: flex;
  min-width: 0;
  flex: 1 1 auto;
  flex-direction: column;
  gap: 4px;
}

.article-path-entry__eyebrow {
  color: var(--color-emphasized);
  font-size: 14px;
  line-height: 22px;
  font-weight: var(--font-weight-bold);
}

.article-path-entry__meta {
  color: var(--color-subtle);
  font-size: 14px;
  line-height: 22px;
}

.article-path-entry__chevron {
  display: inline-flex;
  align-items: center;
  color: var(--color-subtle);
}

.article-path-entry-enter-active,
.article-path-entry-leave-active {
  transition:
    transform 220ms ease,
    opacity 220ms ease;
}

.article-path-entry-enter-from,
.article-path-entry-leave-to {
  opacity: 0;
  transform: translateY(18px);
}

@media (min-width: 640px) {
  .survey-page__panel,
  .homepage,
  .quiz-page,
  .suggested-edits-page,
  .article-page {
    max-width: 33rem;
    padding: var(--spacing-150) var(--spacing-100);
  }

  .survey-page__title,
  .homepage__name {
    font-size: 1.625rem;
    line-height: 2.25rem;
  }

  .article-page {
    max-width: 100%;
  }

  .article-page__title-row {
    flex-direction: row;
    align-items: flex-end;
    justify-content: space-between;
  }

  .article-page__layout {
    display: grid;
    grid-template-columns: minmax(0, 1fr) 19rem;
    gap: 24px;
    align-items: start;
  }

  .article-page__content {
    order: 1;
  }

  .article-page__infobox {
    order: 2;
    position: sticky;
    top: 16px;
  }

  .article-path-entry {
    right: 24px;
    bottom: 24px;
    left: auto;
    width: min(420px, calc(100vw - 48px));
  }
}
</style>
