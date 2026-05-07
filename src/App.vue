<script setup>
import { computed, onBeforeUnmount, onMounted, ref, watch } from 'vue';
import { CdxButton, CdxCard, CdxCheckbox, CdxField, CdxIcon, CdxInfoChip, CdxProgressBar, CdxTextInput } from '@wikimedia/codex';
import {
  cdxIconAdd,
  cdxIconArrowNext,
  cdxIconArrowPrevious,
  cdxIconCheck,
  cdxIconCollapse,
  cdxIconClose
  ,
  cdxIconCalendar,
  cdxIconConfigure,
  cdxIconEdit,
  cdxIconExpand
  ,
  cdxIconHeart
} from '@wikimedia/codex-icons';

import ChromeWrapper from './components/ChromeWrapper.vue';
import encyclopediaPillarImage from './assets/five-pillars/encyclopedia.svg';
import neutralPillarImage from './assets/five-pillars/neutral.svg';
import freeContentPillarImage from './assets/five-pillars/free-content.svg';
import civilityPillarImage from './assets/five-pillars/civility.svg';
import noRulesPillarImage from './assets/five-pillars/no-rules.svg';

const currentSurveyStepIndex = ref( 0 );
const currentView = ref( 'signup' );
const resumeSurveyStepKey = ref( 'intent' );
const currentQuizIndex = ref( 0 );
const currentSuggestedEditIndex = ref( 0 );
const recentlyCompletedTaskKey = ref( '' );
const showArticlePathEntry = ref( false );
const suggestedEditsViewport = ref( null );
const videoCompleted = ref( false );
const showVideoBackButton = ref( false );
const hasContributorPath = ref( false );
const showContributorDialog = ref( false );
const showWelcomeSuccessSheet = ref( false );
const showContributorBadgeGlow = ref( false );
const showWikiMinuteModal = ref( false );
const selectedWikiMinuteVideo = ref( null );
const activeTaskKey = ref( 'welcome-survey' );
let articleScrollTimeout = null;

const selectedReason = ref( '' );
const selectedIntent = ref( '' );
const selectedTopics = ref( [] );
const selectedLanguages = ref( [ 'English' ] );
const isInterestedInTranslation = ref( false );
const email = ref( '' );
const selectedQuizAnswers = ref( {} );
const accountUsername = ref( 'CaptainBird' );
const accountPassword = ref( 'Wikipedia1234' );
const accountConfirmPassword = ref( 'Wikipedia1234' );

const userName = 'CaptainBird';
const userInitials = 'Ca';
const welcomeSurveyHeroImage =
  'https://commons.wikimedia.org/wiki/Special:Redirect/file/WYiR_Puzzle_4.gif';
const surveySuccessHeroImage =
  'https://commons.wikimedia.org/wiki/Special:Redirect/file/Wikipedia%2020%20cover%20puzzle%20globe%20blue.png';
const wikiMinuteVideoSource =
  'https://commons.wikimedia.org/wiki/Special:Redirect/file/How_does_Wikipedia_work_%E2%80%93_A_WIKI_MINUTE_16-9.webm';
const featuredWikiMinuteVideoSource =
  'https://commons.wikimedia.org/wiki/Special:Redirect/file/If_volunteers_edit_Wikipedia%2C_how_can_you_trust_it_%E2%80%93_A_WIKI_MINUTE_16-9_%E2%80%93_NO_VO.webm';
const contributorDialogImage =
  'https://commons.wikimedia.org/wiki/Special:Redirect/file/WYiR_Puzzle_5.png';
const isabellaSuggestedEditImage =
  'https://upload.wikimedia.org/wikipedia/commons/thumb/7/7f/Portrait_of_Isabella_I_of_Castile%2C_aged_44.jpg/320px-Portrait_of_Isabella_I_of_Castile%2C_aged_44.jpg';
const parisSuggestedEditImage =
  'https://upload.wikimedia.org/wikipedia/commons/a/a8/Tour_Eiffel_Wikimedia_Commons.jpg';
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

const wikiMinuteVideos = [
  { key: 'how-wikipedia-works', title: 'How does Wikipedia work?', source: wikiMinuteVideoSource, watched: true },
  { key: 'diversity', title: 'Does the content on Wikipedia reflect the world’s diversity?', source: wikiMinuteVideoSource },
  { key: 'trust', title: 'Can you trust what’s on Wikipedia?', source: wikiMinuteVideoSource },
  { key: 'unique-article', title: 'What makes a Wikipedia article unique?', source: wikiMinuteVideoSource },
  { key: 'fundraising', title: 'Why do you see fundraising messages on Wikipedia?', source: wikiMinuteVideoSource },
  { key: 'wmf', title: 'What does the Wikimedia Foundation do?', source: wikiMinuteVideoSource },
  { key: 'movement', title: 'What is the Wikimedia free knowledge movement?', source: wikiMinuteVideoSource },
  { key: 'projects', title: 'What free knowledge projects does the Wikimedia Foundation support?', source: wikiMinuteVideoSource },
  { key: 'join-movement', title: 'How can you join the Wikimedia free knowledge movement?', source: wikiMinuteVideoSource },
  { key: 'misinformation', title: 'How is misinformation addressed on Wikipedia?', source: wikiMinuteVideoSource },
  { key: 'privacy', title: 'How does Wikipedia protect readers’ privacy?', source: wikiMinuteVideoSource },
  { key: 'content-governance', title: 'Who is in charge of content on Wikipedia?', source: wikiMinuteVideoSource },
  { key: 'social-media', title: 'What makes Wikipedia different from social media platforms?', source: wikiMinuteVideoSource },
  { key: 'ai-role', title: 'What is the role of Wikipedia in the age of AI?', source: wikiMinuteVideoSource },
  { key: 'volunteers-trust', title: 'If volunteers edit Wikipedia, how can you trust it?', source: featuredWikiMinuteVideoSource },
  { key: 'run-wikipedia', title: 'What does it take to run Wikipedia?', source: wikiMinuteVideoSource }
];

const intentOptions = [
  {
    key: 'read',
    title: 'Read and explore',
    description: ''
  },
  {
    key: 'edit',
    title: 'Edit and contribute',
    description: ''
  },
  {
    key: 'both',
    title: 'A bit of both',
    description: ''
  }
];

const reasonOptions = [
  {
    key: 'never',
    title: "I've never edited",
    description: ''
  },
  {
    key: 'couple',
    title: "I've made a couple of edits",
    description: ''
  },
  {
    key: 'unknown',
    title: 'I don’t know',
    description: ''
  }
];

const topicCategories = [
  {
    key: 'culture',
    title: 'Culture',
    topics: [
      'Architecture',
      'Art',
      'Comics & anime',
      'Entertainment',
      'Fashion',
      'Literature',
      'Music',
      'Performing arts',
      'Sports',
      'TV/Film',
      'Video games'
    ]
  },
  {
    key: 'history-society',
    title: 'History and Society',
    topics: [
      'Biography (all)',
      'Biography (women)',
      'Business & economics',
      'Education',
      'Food & drink',
      'History',
      'Military & warfare',
      'Philosophy & religion',
      'Politics & government',
      'Society',
      'Transportation'
    ]
  },
  {
    key: 'stem',
    title: 'Science, Technology, and Math',
    topics: [
      'Biology',
      'Chemistry',
      'Computers & internet',
      'Earth & environment',
      'Engineering',
      'General science',
      'Mathematics',
      'Medicine & health',
      'Physics',
      'Technology'
    ]
  },
  {
    key: 'regions',
    title: 'Regions',
    topics: [
      'Africa',
      'Asia',
      'Central America',
      'Europe',
      'North America',
      'Oceania',
      'South America'
    ]
  }
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
        text: 'Yes, it\'s a well-known fact.'
      },
      {
        key: 'b',
        label: 'B',
        text: 'No, it\'s an opinion.'
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
        text: 'No, it\'s an opinion.'
      },
      {
        key: 'b',
        label: 'B',
        text: 'Yes, it\'s a well-known fact.'
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
        text: 'Yes, if it\'s accurate and well-written, it improves the article.'
      },
      {
        key: 'b',
        label: 'B',
        text: 'No, copying copyrighted text violates Wikipedia\'s policy.'
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
        text: 'No, good content shouldn\'t be removed over formatting.'
      },
      {
        key: 'c',
        label: 'C',
        text: 'It depends on how wrong the formatting was.'
      }
    ]
  }
];

function createBeginnerTasks() {
  return [
    {
      key: 'welcome-survey',
      title: 'Complete Welcome Survey',
      description: 'Respond four quick questions to personalize your experience',
      duration: '2 min',
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
      key: 'first-edit',
      title: 'First suggested edit',
      description: 'A small and easy edit picked for you',
      duration: '2 min',
      buttonLabel: 'See suggested edits',
      action: 'edit',
      completed: false
    },
    {
      key: 'five-pillars',
      title: 'Five Pillars quiz',
      description: 'Review the core principles that guide editing on Wikipedia',
      duration: '5 min',
      buttonLabel: 'Review pillars',
      action: 'quiz',
      completed: false
    },
    {
      key: 'enrich-article',
      title: 'Enrich an article',
      description: 'Choose one way to improve an article: citation, image, or link',
      duration: '10 min',
      buttonLabel: 'Enrich article',
      action: 'enrich',
      completed: false
    }
  ];
}

function createContributorTasks() {
  return [
    {
      key: 'create-user-page',
      title: 'Create user page',
      description: 'Description lorem ipsum',
      duration: '10 min',
      buttonLabel: 'Create page',
      action: 'contributor',
      completed: false
    },
    {
      key: 'talk-page-edit',
      title: 'First talk page edit',
      description: 'Description lorem ipsum',
      duration: '5 min',
      buttonLabel: 'Open talk page',
      action: 'contributor',
      completed: false
    },
    {
      key: 'wiki-event',
      title: 'Join a wiki event',
      description: 'Description lorem ipsum',
      duration: '5 min',
      buttonLabel: 'Find event',
      action: 'contributor',
      completed: false
    },
    {
      key: 'ten-suggestions',
      title: 'Complete 10 suggestions',
      description: 'Description lorem ipsum',
      duration: '15 min',
      buttonLabel: 'See suggestions',
      action: 'contributor',
      completed: false
    },
    {
      key: 'three-article-edits',
      title: 'Complete 3 article edits',
      description: 'Description lorem ipsum',
      duration: '30 min',
      buttonLabel: 'Start editing',
      action: 'contributor',
      completed: false
    }
  ];
}

const progressionTasks = ref( createBeginnerTasks() );

const editsCount = ref( 0 );
const thanksCount = ref( 0 );
const streakCount = ref( 0 );
const isProgressionExpanded = ref( false );
const suggestedEditTemplates = {
  convertReference: {
    taskTitle: 'Convert reference',
    taskDescription:
      'This reference is missing details. Help readers understand where the information is coming from by converting this into a formatted reference. After converting it, check the accuracy of the details that the tool has added, and remove anything that is wrong or confusing.',
    primaryLabel: 'Convert',
    secondaryLabel: 'Dismiss',
    successTitle: 'Converted the reference',
    successMessage: 'The source details are easier to verify now.'
  },
  specificPage: {
    taskTitle: 'Link to a more specific page',
    taskDescription:
      'This link points to a disambiguation page. Help readers find the right topic by linking to a more specific page.',
    primaryLabel: 'Update link',
    secondaryLabel: 'Dismiss',
    successTitle: 'Updated the link',
    successMessage: 'Readers will now land on the intended topic.'
  },
  duplicateLink: {
    taskTitle: 'Remove duplicated link',
    taskDescription:
      'This link appears more than once in this section. Help readers navigate more easily by removing repeated links.',
    primaryLabel: 'Remove link',
    secondaryLabel: 'Dismiss',
    successTitle: 'Removed duplicate link',
    successMessage: 'Thank you for improving Wikipedia.'
  },
  externalLink: {
    taskTitle: 'Remove external link',
    taskDescription:
      'This link points to an external website. Help readers stay focused on the content by removing this link, moving it to the “External links” section, or converting it into a citation if appropriate.',
    primaryLabel: 'Remove link',
    secondaryLabel: 'Dismiss',
    successTitle: 'Removed the external link',
    successMessage: 'That section is now more focused on encyclopedic content.'
  },
  headingLevel: {
    taskTitle: 'Adjust heading level',
    taskDescription:
      'This heading level may not follow the correct sequential structure. Help readers navigate the content more easily by using the correct heading level.',
    primaryLabel: 'Adjust heading',
    secondaryLabel: 'Dismiss',
    successTitle: 'Adjusted the heading',
    successMessage: 'The article structure is now clearer.'
  },
  imageCaption: {
    taskTitle: 'Add image caption',
    taskDescription:
      'This image does not have a caption. Help readers understand why the image is relevant by adding a short caption.',
    primaryLabel: 'Add caption',
    secondaryLabel: 'Dismiss',
    successTitle: 'Added a caption',
    successMessage: 'The image is easier to understand in context.'
  },
  englishSpelling: {
    taskTitle: 'Change English spelling',
    taskDescription:
      'This word uses a different variety of English than the one used in the rest of this article. Help readers by changing the spelling to match the rest of the article.',
    primaryLabel: 'Replace',
    secondaryLabel: 'Dismiss',
    successTitle: 'Updated the spelling',
    successMessage: 'Thanks for keeping the article consistent.'
  },
  redirectLink: {
    taskTitle: 'Redirect link',
    taskDescription:
      'This link points to a redirect page. Help readers reach the intended article directly by linking to the final target.',
    primaryLabel: 'Update link',
    secondaryLabel: 'Dismiss',
    successTitle: 'Updated the redirect',
    successMessage: 'The link now points directly to the article.'
  },
  yearLink: {
    taskTitle: 'Fix year link',
    taskDescription:
      'This link points to a different year than the one shown in the text. Help make the article more accurate by updating the displayed year and linked year to match.',
    primaryLabel: 'Use correct year',
    secondaryLabel: 'Dismiss',
    successTitle: 'Fixed the year link',
    successMessage: 'The linked year now matches the text.'
  },
  cliche: {
    taskTitle: 'Remove clichés and idioms',
    taskDescription:
      'This word uses a cliché or idiom. Help readers understand the content more clearly by using direct and literal language instead.',
    primaryLabel: 'Remove word',
    secondaryLabel: 'Dismiss',
    successTitle: 'Revised the wording',
    successMessage: 'The sentence is clearer and more neutral now.'
  },
  relativeTime: {
    taskTitle: 'Edit relative time references',
    taskDescription:
      'This text uses a relative time reference. Help readers avoid confusion by replacing it with a specific date or time.',
    primaryLabel: 'Edit text',
    secondaryLabel: 'Dismiss',
    successTitle: 'Updated the time reference',
    successMessage: 'The wording will age better over time.'
  },
  editorialLanguage: {
    taskTitle: 'Revise editorial language',
    taskDescription:
      'This wording adds opinion rather than stating facts. Help readers trust the article by revising the text to keep an impartial tone.',
    primaryLabel: 'Revise',
    secondaryLabel: 'Dismiss',
    successTitle: 'Revised the language',
    successMessage: 'The tone is now more neutral.'
  },
  changeTerm: {
    taskTitle: 'Change term',
    taskDescription:
      'This article uses the term “aircraft” instead of “airplane”. Help readers read the article consistently by updating the wording.',
    primaryLabel: 'Change word',
    secondaryLabel: 'Dismiss',
    successTitle: 'Updated the term',
    successMessage: 'The terminology is now consistent.'
  },
  aiContent: {
    taskTitle: 'Potential AI-generated content',
    taskDescription:
      'This text may include AI-generated content. Help readers trust the article by removing any AI content or rewriting any inaccurate, unverifiable, or unencyclopedic information.',
    primaryLabel: 'Revise text',
    secondaryLabel: 'Dismiss',
    successTitle: 'Reviewed the content',
    successMessage: 'The article now reads more reliably.'
  }
};

function buildSuggestedEdit( config ) {
  return {
    ...suggestedEditTemplates[ config.template ],
    key: config.key,
    sequenceNumber: config.sequenceNumber,
    articleTitle: config.articleTitle,
    articleDescription: config.articleDescription,
    articleImage: config.articleImage,
    articleSnippetHtml: config.articleSnippetHtml,
    resolvedSnippetHtml: config.resolvedSnippetHtml || config.articleSnippetHtml,
    question: config.question || '',
    topicTags: config.topicTags || []
  };
}

const suggestedEdits = ref( [
  buildSuggestedEdit( {
    key: 'isabella-duplicate-link',
    sequenceNumber: 1,
    template: 'duplicateLink',
    articleTitle: 'Isabella I of Castile',
    articleDescription: 'Queen of Castile and León',
    articleImage: isabellaSuggestedEditImage,
    articleSnippetHtml:
      'Isabella and Ferdinand are known for being the first monarchs to be referred to as the queen and king of <a href="#" class="suggested-edit-card__link"><span class="suggested-edit-card__highlight">Spain</span></a>, respectively. Their actions included completion of the <a href="#" class="suggested-edit-card__link">Reconquista</a>, the <a href="#" class="suggested-edit-card__link">Alhambra Decree</a> which ordered the <a href="#" class="suggested-edit-card__link">mass expulsion of Jews</a> from <a href="#" class="suggested-edit-card__link"><span class="suggested-edit-card__highlight">Spain</span></a>, initiating the <a href="#" class="suggested-edit-card__link">Spanish Inquisition</a>, financing Christopher Columbus’s 1492 voyage to the New World.',
    resolvedSnippetHtml:
      'Isabella and Ferdinand are known for being the first monarchs to be referred to as the queen and king of <a href="#" class="suggested-edit-card__link">Spain</a>, respectively. Their actions included completion of the <a href="#" class="suggested-edit-card__link">Reconquista</a>, the <a href="#" class="suggested-edit-card__link">Alhambra Decree</a> which ordered the <a href="#" class="suggested-edit-card__link">mass expulsion of Jews</a> from Spain, initiating the <a href="#" class="suggested-edit-card__link">Spanish Inquisition</a>, financing Christopher Columbus’s 1492 voyage to the New World.'
    ,
    topicTags: [ 'History', 'Biography (women)', 'Europe' ]
  } ),
  buildSuggestedEdit( {
    key: 'isabella-uk-spelling',
    sequenceNumber: 2,
    template: 'englishSpelling',
    articleTitle: 'Isabella I of Castile',
    articleDescription: 'Queen of Castile and León',
    articleImage: isabellaSuggestedEditImage,
    articleSnippetHtml:
      'Girón. Desiring to depose Henry and establish Infante Alfonso on the throne, Pacheco and his followers circulated <span class="suggested-edit-card__highlight">rumors</span> that Infanta Joanna was actually the child of Beltrán de la Cueva.',
    resolvedSnippetHtml:
      'Girón. Desiring to depose Henry and establish Infante Alfonso on the throne, Pacheco and his followers circulated <span class="suggested-edit-card__highlight">rumours</span> that Infanta Joanna was actually the child of Beltrán de la Cueva.',
    question: 'Replace “rumors” with “rumours”?',
    topicTags: [ 'History', 'Biography (women)', 'Europe' ]
  } ),
  buildSuggestedEdit( {
    key: 'paris-specific-link',
    sequenceNumber: 3,
    template: 'specificPage',
    articleTitle: 'Paris',
    articleDescription: 'Capital and largest city of France',
    articleImage: parisSuggestedEditImage,
    articleSnippetHtml:
      'Paris developed from a Celtic settlement on the <a href="#" class="suggested-edit-card__link"><span class="suggested-edit-card__highlight">Cité</span></a> into a major political, cultural, and commercial center of Europe. Over centuries the city expanded on both banks of the Seine.',
    resolvedSnippetHtml:
      'Paris developed from a Celtic settlement on the <a href="#" class="suggested-edit-card__link">Île de la Cité</a> into a major political, cultural, and commercial center of Europe. Over centuries the city expanded on both banks of the Seine.'
    ,
    topicTags: [ 'Architecture', 'Europe', 'History' ]
  } ),
  buildSuggestedEdit( {
    key: 'apollo-convert-ref',
    sequenceNumber: 4,
    template: 'convertReference',
    articleTitle: 'Apollo 11',
    articleDescription: 'First crewed Moon landing mission',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/9/9c/Apollo_11_insignia.png',
    articleSnippetHtml:
      'Apollo 11 landed the first two people on the Moon in July 1969.<sup><span class="suggested-edit-card__highlight">[book]</span></sup>',
    question: 'Convert this source into a full reference?'
  } ),
  buildSuggestedEdit( {
    key: 'jane-external-link',
    sequenceNumber: 5,
    template: 'externalLink',
    articleTitle: 'Jane Austen',
    articleDescription: 'English novelist',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/c/cc/CassandraAusten-JaneAusten%28c.1810%29_hires.jpg',
    articleSnippetHtml:
      'Austen’s novels critique the British landed gentry and often focus on women’s dependence on marriage. See also <a href="#" class="suggested-edit-card__link"><span class="suggested-edit-card__highlight">this fan website</span></a> for a character guide.'
  } ),
  buildSuggestedEdit( {
    key: 'python-heading',
    sequenceNumber: 6,
    template: 'headingLevel',
    articleTitle: 'Python (programming language)',
    articleDescription: 'High-level programming language',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/c/c3/Python-logo-notext.svg',
    articleSnippetHtml:
      '<strong>== History ==</strong><br><strong><span class="suggested-edit-card__highlight">==== Design philosophy ====</span></strong><br>Python emphasizes code readability and a large standard library.'
  } ),
  buildSuggestedEdit( {
    key: 'hubble-caption',
    sequenceNumber: 7,
    template: 'imageCaption',
    articleTitle: 'Hubble Space Telescope',
    articleDescription: 'Space telescope in low Earth orbit',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/3/3f/HST-SM4.jpeg',
    articleSnippetHtml:
      '<span class="suggested-edit-card__highlight">[Image inserted here with no caption]</span><br>The telescope was launched in 1990 and remains one of the most productive scientific instruments ever built.'
  } ),
  buildSuggestedEdit( {
    key: 'newyork-redirect',
    sequenceNumber: 8,
    template: 'redirectLink',
    articleTitle: 'New York City',
    articleDescription: 'Most populous city in the United States',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/4/47/New_york_times_square-terabass.jpg',
    articleSnippetHtml:
      'The city is located at the southern tip of <a href="#" class="suggested-edit-card__link"><span class="suggested-edit-card__highlight">Manhattan Island redirect</span></a> and anchors the largest metropolitan area in the United States.',
    resolvedSnippetHtml:
      'The city is located at the southern tip of <a href="#" class="suggested-edit-card__link">Manhattan</a> and anchors the largest metropolitan area in the United States.'
  } ),
  buildSuggestedEdit( {
    key: 'wwii-year-link',
    sequenceNumber: 9,
    template: 'yearLink',
    articleTitle: 'World War II',
    articleDescription: 'Global war from 1939 to 1945',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/a/ad/Nagasakibomb.jpg',
    articleSnippetHtml:
      'The conflict ended in <a href="#" class="suggested-edit-card__link"><span class="suggested-edit-card__highlight">1944</span></a> after the surrender of Japan in 1945.',
    resolvedSnippetHtml:
      'The conflict ended in <a href="#" class="suggested-edit-card__link">1945</a> after the surrender of Japan in 1945.'
  } ),
  buildSuggestedEdit( {
    key: 'einstein-cliche',
    sequenceNumber: 10,
    template: 'cliche',
    articleTitle: 'Albert Einstein',
    articleDescription: 'German-born theoretical physicist',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/d/d3/Albert_Einstein_Head.jpg',
    articleSnippetHtml:
      'Einstein’s 1905 papers <span class="suggested-edit-card__highlight">turned physics upside down</span> and changed the field forever.',
    resolvedSnippetHtml:
      'Einstein’s 1905 papers significantly changed the development of modern physics.'
  } ),
  buildSuggestedEdit( {
    key: 'olympics-relative-time',
    sequenceNumber: 11,
    template: 'relativeTime',
    articleTitle: '2024 Summer Olympics',
    articleDescription: 'International multi-sport event in Paris',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/5/57/Paris_2024_logo.svg',
    articleSnippetHtml:
      'The event concluded <span class="suggested-edit-card__highlight">last summer</span> after two weeks of competition in Paris.',
    resolvedSnippetHtml:
      'The event concluded on 11 August 2024 after two weeks of competition in Paris.'
  } ),
  buildSuggestedEdit( {
    key: 'tesla-editorial',
    sequenceNumber: 12,
    template: 'editorialLanguage',
    articleTitle: 'Nikola Tesla',
    articleDescription: 'Inventor and electrical engineer',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/d/d4/N.Tesla.JPG',
    articleSnippetHtml:
      'Tesla’s <span class="suggested-edit-card__highlight">visionary genius</span> made him one of the most brilliant inventors in history.',
    resolvedSnippetHtml:
      'Tesla is widely known for his work in electrical engineering and wireless communication.'
  } ),
  buildSuggestedEdit( {
    key: 'wright-change-term',
    sequenceNumber: 13,
    template: 'changeTerm',
    articleTitle: 'Wright brothers',
    articleDescription: 'American aviation pioneers',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/a/af/Orville_Wright_1905-crop.jpg',
    articleSnippetHtml:
      'Their work on powered <span class="suggested-edit-card__highlight">aircraft</span> helped define the early history of human flight.',
    resolvedSnippetHtml:
      'Their work on powered airplane design helped define the early history of human flight.'
  } ),
  buildSuggestedEdit( {
    key: 'mlk-ai',
    sequenceNumber: 14,
    template: 'aiContent',
    articleTitle: 'Martin Luther King Jr.',
    articleDescription: 'American civil rights leader',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/0/05/Martin_Luther_King%2C_Jr..jpg',
    articleSnippetHtml:
      'King was an <span class="suggested-edit-card__highlight">inspirational figure</span> whose universally beloved speeches flawlessly transformed the soul of a nation in every possible way.'
  } ),
  buildSuggestedEdit( {
    key: 'rome-specific-link',
    sequenceNumber: 15,
    template: 'specificPage',
    articleTitle: 'Rome',
    articleDescription: 'Capital city of Italy',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/0/00/Altare_della_Patria_%28Rome%29.jpg',
    articleSnippetHtml:
      'The city grew around the <a href="#" class="suggested-edit-card__link"><span class="suggested-edit-card__highlight">Forum</span></a> and became the center of a vast empire.',
    resolvedSnippetHtml:
      'The city grew around the <a href="#" class="suggested-edit-card__link">Roman Forum</a> and became the center of a vast empire.'
  } ),
  buildSuggestedEdit( {
    key: 'turing-convert-ref',
    sequenceNumber: 16,
    template: 'convertReference',
    articleTitle: 'Alan Turing',
    articleDescription: 'English mathematician and computer scientist',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/a/a1/Alan_Turing_Aged_16.jpg',
    articleSnippetHtml:
      'Turing played a key role in cracking encrypted German naval communications during World War II.<sup><span class="suggested-edit-card__highlight">[news]</span></sup>'
  } ),
  buildSuggestedEdit( {
    key: 'moon-duplicate-link',
    sequenceNumber: 17,
    template: 'duplicateLink',
    articleTitle: 'Moon',
    articleDescription: 'Earth’s only natural satellite',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/e/e1/FullMoon2010.jpg',
    articleSnippetHtml:
      'The <a href="#" class="suggested-edit-card__link"><span class="suggested-edit-card__highlight">Moon</span></a> is Earth’s only natural satellite. The <a href="#" class="suggested-edit-card__link"><span class="suggested-edit-card__highlight">Moon</span></a> influences tides and stabilizes Earth’s axial tilt.',
    resolvedSnippetHtml:
      'The <a href="#" class="suggested-edit-card__link">Moon</a> is Earth’s only natural satellite. It influences tides and stabilizes Earth’s axial tilt.'
  } ),
  buildSuggestedEdit( {
    key: 'london-external-link',
    sequenceNumber: 18,
    template: 'externalLink',
    articleTitle: 'London',
    articleDescription: 'Capital city of England and the United Kingdom',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/c/cd/London_Montage_L.jpg',
    articleSnippetHtml:
      'London is one of the world’s leading financial centers. For restaurant recommendations, see <a href="#" class="suggested-edit-card__link"><span class="suggested-edit-card__highlight">this travel blog</span></a>.'
  } ),
  buildSuggestedEdit( {
    key: 'mars-heading',
    sequenceNumber: 19,
    template: 'headingLevel',
    articleTitle: 'Mars',
    articleDescription: 'Fourth planet from the Sun',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/0/02/OSIRIS_Mars_true_color.jpg',
    articleSnippetHtml:
      '<strong>== Geology ==</strong><br><strong><span class="suggested-edit-card__highlight">==== Surface features ====</span></strong><br>Mars has volcanoes, valleys, deserts, and polar ice caps.'
  } ),
  buildSuggestedEdit( {
    key: 'notredame-caption',
    sequenceNumber: 20,
    template: 'imageCaption',
    articleTitle: 'Notre-Dame de Paris',
    articleDescription: 'Medieval Catholic cathedral in Paris',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/a/a6/Notre-Dame_de_Paris_2013-07.JPG',
    articleSnippetHtml:
      '<span class="suggested-edit-card__highlight">[Image inserted here with no caption]</span><br>The cathedral is considered one of the finest examples of French Gothic architecture.'
  } ),
  buildSuggestedEdit( {
    key: 'canada-spelling',
    sequenceNumber: 21,
    template: 'englishSpelling',
    articleTitle: 'Canada',
    articleDescription: 'Country in North America',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/c/cf/Flag_of_Canada.svg',
    articleSnippetHtml:
      'The country’s national <span class="suggested-edit-card__highlight">color</span> symbolism often appears in official documents.',
    resolvedSnippetHtml:
      'The country’s national <span class="suggested-edit-card__highlight">colour</span> symbolism often appears in official documents.',
    question: 'Replace “color” with “colour”?'
  } ),
  buildSuggestedEdit( {
    key: 'berlin-redirect',
    sequenceNumber: 22,
    template: 'redirectLink',
    articleTitle: 'Berlin',
    articleDescription: 'Capital and largest city of Germany',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/a/a5/Berlin_Skyline_Fernsehturm_02.jpg',
    articleSnippetHtml:
      'Berlin is built along the banks of the <a href="#" class="suggested-edit-card__link"><span class="suggested-edit-card__highlight">River Spree redirect</span></a> and is one of Europe’s most populous cities.',
    resolvedSnippetHtml:
      'Berlin is built along the banks of the <a href="#" class="suggested-edit-card__link">Spree</a> and is one of Europe’s most populous cities.'
  } ),
  buildSuggestedEdit( {
    key: 'internet-year-link',
    sequenceNumber: 23,
    template: 'yearLink',
    articleTitle: 'Internet',
    articleDescription: 'Global system of computer networks',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/4/4e/Internet_map_1024.jpg',
    articleSnippetHtml:
      'Commercial internet service providers began to emerge in <a href="#" class="suggested-edit-card__link"><span class="suggested-edit-card__highlight">1988</span></a>, but the text says 1989.',
    resolvedSnippetHtml:
      'Commercial internet service providers began to emerge in <a href="#" class="suggested-edit-card__link">1989</a>.'
  } ),
  buildSuggestedEdit( {
    key: 'oceans-cliche',
    sequenceNumber: 24,
    template: 'cliche',
    articleTitle: 'Pacific Ocean',
    articleDescription: 'Largest and deepest ocean on Earth',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/6/6f/Pacific_Ocean_-_en.png',
    articleSnippetHtml:
      'The Pacific plays <span class="suggested-edit-card__highlight">a huge role</span> in global climate and trade routes.',
    resolvedSnippetHtml:
      'The Pacific has an important role in global climate and trade routes.'
  } ),
  buildSuggestedEdit( {
    key: 'everest-relative-time',
    sequenceNumber: 25,
    template: 'relativeTime',
    articleTitle: 'Mount Everest',
    articleDescription: 'Earth’s highest mountain above sea level',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/d/d1/Mount_Everest_as_seen_from_Drukair2_PLW_edit.jpg',
    articleSnippetHtml:
      'The climbing season reopened <span class="suggested-edit-card__highlight">last year</span> after weather disruptions.',
    resolvedSnippetHtml:
      'The climbing season reopened in 2025 after weather disruptions.'
  } ),
  buildSuggestedEdit( {
    key: 'shakespeare-editorial',
    sequenceNumber: 26,
    template: 'editorialLanguage',
    articleTitle: 'William Shakespeare',
    articleDescription: 'English playwright and poet',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/a/a2/Shakespeare.jpg',
    articleSnippetHtml:
      'Shakespeare wrote <span class="suggested-edit-card__highlight">the most extraordinary plays</span> in the history of the English language.',
    resolvedSnippetHtml:
      'Shakespeare wrote plays that are among the most studied works in the English language.'
  } ),
  buildSuggestedEdit( {
    key: 'boeing-change-term',
    sequenceNumber: 27,
    template: 'changeTerm',
    articleTitle: 'Boeing 747',
    articleDescription: 'American wide-body commercial jet airliner',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/2/2e/B747_ANA_JA8961.jpg',
    articleSnippetHtml:
      'The aircraft became one of the best-known long-haul passenger aircraft in aviation history.',
    resolvedSnippetHtml:
      'The airplane became one of the best-known long-haul passenger airplanes in aviation history.'
  } ),
  buildSuggestedEdit( {
    key: 'chatgpt-ai',
    sequenceNumber: 28,
    template: 'aiContent',
    articleTitle: 'ChatGPT',
    articleDescription: 'Artificial intelligence chatbot by OpenAI',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/0/04/ChatGPT_logo.svg',
    articleSnippetHtml:
      'The service instantly revolutionized every field of knowledge and produced <span class="suggested-edit-card__highlight">perfectly accurate answers</span> for nearly all users.'
  } ),
  buildSuggestedEdit( {
    key: 'newton-convert-ref',
    sequenceNumber: 29,
    template: 'convertReference',
    articleTitle: 'Isaac Newton',
    articleDescription: 'English mathematician, physicist, and astronomer',
    articleImage: 'https://upload.wikimedia.org/wikipedia/commons/d/d4/Sir_Isaac_Newton_%281643-1727%29.jpg',
    articleSnippetHtml:
      'Newton formulated the laws of motion and universal gravitation that dominated scientific thought for centuries.<sup><span class="suggested-edit-card__highlight">[website]</span></sup>'
  } )
] );

const surveySteps = computed( () => {
  const flow = [ 'welcome', 'intent' ];

  if ( selectedIntent.value === 'edit' || selectedIntent.value === 'both' || !selectedIntent.value ) {
    flow.push( 'reason' );
  }

  flow.push( 'topics', 'languages', 'email' );
  return flow;
} );
const currentSurveyStep = computed( () => surveySteps.value[ currentSurveyStepIndex.value ] );
const totalSurveyQuestionCount = computed( () => surveySteps.value.length - 1 );
const currentQuizStep = computed( () => quizSteps[ currentQuizIndex.value ] );
const completedTaskCount = computed(
  () => progressionTasks.value.filter( ( task ) => task.completed ).length
);
const firstIncompleteTaskIndex = computed(
  () => progressionTasks.value.findIndex( ( task ) => !task.completed )
);
const currentTaskIndex = computed( () => {
  const activeIndex = progressionTasks.value.findIndex(
    ( task ) => task.key === activeTaskKey.value && !task.completed
  );

  if ( activeIndex >= 0 ) {
    return activeIndex;
  }

  return firstIncompleteTaskIndex.value;
} );
const currentTaskNumber = computed( () => currentTaskIndex.value + 1 );
const activeProgressTask = computed(
  () => ( currentTaskIndex.value >= 0 ? progressionTasks.value[ currentTaskIndex.value ] : null )
);
const profileLevelLabel = computed( () => ( hasContributorPath.value ? 'Contributor' : 'Beginner' ) );
const profileLevelStatus = computed( () => ( hasContributorPath.value ? 'progressive' : 'subtle' ) );
const profileJoinedText = computed( () => ( hasContributorPath.value ? 'Joined 7 days ago' : 'Joined today' ) );
const shouldCollapseProgression = computed( () => false );
const visibleTasks = computed( () => progressionTasks.value );
const currentQuizAnswer = computed(
  () => selectedQuizAnswers.value[ currentQuizStep.value.key ] || ''
);
const currentQuizAnswerIsCorrect = computed(
  () => currentQuizAnswer.value === currentQuizStep.value.correctKey
);
const rankedSuggestedEdits = computed( () =>
  suggestedEdits.value
    .map( ( suggestion, index ) => ( {
      suggestion,
      index,
      score: suggestion.topicTags.filter( ( tag ) => selectedTopics.value.includes( tag ) ).length
    } ) )
    .sort( ( a, b ) => {
      if ( b.score !== a.score ) {
        return b.score - a.score;
      }

      return a.index - b.index;
    } )
    .map( ( item ) => item.suggestion )
);
const remainingSuggestedEdits = computed(
  () => rankedSuggestedEdits.value.filter( ( suggestion ) => !suggestion.resolved )
);
const renderedSuggestedEdits = computed( () => {
  const items = remainingSuggestedEdits.value;

  if ( items.length <= 1 ) {
    return items.map( ( suggestion ) => ( {
      key: suggestion.key,
      suggestion
    } ) );
  }

  return [
    {
      key: `${items[ items.length - 1 ].key}-clone-start`,
      suggestion: items[ items.length - 1 ],
      isClone: true
    },
    ...items.map( ( suggestion ) => ( {
      key: suggestion.key,
      suggestion
    } ) ),
    {
      key: `${items[ 0 ].key}-clone-end`,
      suggestion: items[ 0 ],
      isClone: true
    }
  ];
} );
const currentSuggestedEdit = computed(
  () => remainingSuggestedEdits.value[ currentSuggestedEditIndex.value ] || null
);
const currentSuggestedEditSequenceNumber = computed(
  () => Math.min( currentSuggestedEditIndex.value + 1, remainingSuggestedEdits.value.length || 1 )
);
const hasCompletedWikiMinuteTask = computed(
  () => hasContributorPath.value || progressionTasks.value.find( ( task ) => task.key === 'wiki-minute' )?.completed
);
const hasCompletedFirstEditTask = computed(
  () => hasContributorPath.value || progressionTasks.value.find( ( task ) => task.key === 'first-edit' )?.completed
);
const featuredWikiMinuteVideo = computed(
  () => wikiMinuteVideos.find( ( video ) => video.key === 'volunteers-trust' ) || wikiMinuteVideos[ 0 ]
);
const homepageSuggestedEditsPreview = computed( () =>
  rankedSuggestedEdits.value
    .slice( 0, 8 )
);
const currentSurveyQuestionNumber = computed( () =>
  currentSurveyStepIndex.value > 0 ? currentSurveyStepIndex.value : 0
);
const shouldShowChromeFooter = computed(
  () => ![ 'signup', 'survey', 'survey-success', 'quiz', 'suggested-edits', 'video', 'wiki-minute-library' ].includes( currentView.value )
);
const shouldShowChromeSiteHeader = computed(
  () => ![ 'suggested-edits', 'video', 'wiki-minute-library' ].includes( currentView.value )
);
const chromeHeaderMode = computed(
  () => ( currentView.value === 'signup' ? 'account-creation' : 'default' )
);

const surveyProgressValue = computed( () => {
  if ( currentSurveyStep.value === 'welcome' ) {
    return 0;
  }

  return Math.round( ( currentSurveyQuestionNumber.value / totalSurveyQuestionCount.value ) * 100 );
} );

const canAdvanceSurvey = computed( () => {
  switch ( currentSurveyStep.value ) {
    case 'intent':
      return Boolean( selectedIntent.value );
    case 'reason':
      return Boolean( selectedReason.value );
    case 'topics':
      return selectedTopics.value.length >= 4;
    case 'languages':
      return selectedLanguages.value.length > 0;
    case 'email':
      return true;
    default:
      return false;
  }
} );

watch(
  currentView,
  ( nextView ) => {
    if ( nextView === 'home' ) {
      isProgressionExpanded.value = !shouldCollapseProgression.value;
      triggerProfileBadgeGlow();
    }

    if ( nextView === 'article' ) {
      showArticlePathEntry.value = false;
      if ( articleScrollTimeout ) {
        clearTimeout( articleScrollTimeout );
      }
      articleScrollTimeout = setTimeout( () => {
        if ( currentView.value === 'article' && activeProgressTask.value ) {
          showArticlePathEntry.value = true;
        }
      }, 1000 );
    } else {
      showArticlePathEntry.value = false;
      if ( articleScrollTimeout ) {
        clearTimeout( articleScrollTimeout );
        articleScrollTimeout = null;
      }
    }
  },
  { immediate: true }
);

watch(
  progressionTasks,
  ( nextTasks ) => {
    const currentActiveTask = nextTasks.find(
      ( task ) => task.key === activeTaskKey.value && !task.completed
    );

    if ( currentActiveTask ) {
      return;
    }

    const nextIncompleteTask = nextTasks.find( ( task ) => !task.completed );
    activeTaskKey.value = nextIncompleteTask?.key || '';
  },
  { deep: true, immediate: true }
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
  if ( currentSurveyStepIndex.value < surveySteps.value.length - 1 ) {
    currentSurveyStepIndex.value += 1;
    resumeSurveyStepKey.value = surveySteps.value[ currentSurveyStepIndex.value ];
  }
}

function goToWelcomeSurvey() {
  currentSurveyStepIndex.value = 0;
  currentView.value = 'survey';
}

function goToPreviousSurveyStep() {
  if ( currentSurveyStepIndex.value > 1 ) {
    currentSurveyStepIndex.value -= 1;
    resumeSurveyStepKey.value = surveySteps.value[ currentSurveyStepIndex.value ];
  }
}

function toggleTopic( key ) {
  if ( selectedTopics.value.includes( key ) ) {
    selectedTopics.value = selectedTopics.value.filter( ( item ) => item !== key );
    return;
  }

  selectedTopics.value = [ ...selectedTopics.value, key ];
}

function toggleGroupedTopic( topic ) {
  toggleTopic( topic );
}

function isTopicCategoryFullySelected( category ) {
  return category.topics.every( ( topic ) => selectedTopics.value.includes( topic ) );
}

function selectAllTopicsInCategory( category ) {
  const nextTopics = new Set( selectedTopics.value );

  category.topics.forEach( ( topic ) => {
    nextTopics.add( topic );
  } );

  selectedTopics.value = [ ...nextTopics ];
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
  if ( currentSurveyStep.value !== 'welcome' ) {
    resumeSurveyStepKey.value = currentSurveyStep.value;
  }
  currentView.value = 'home';
}

function completeSurvey() {
  markTaskCompleted( 'welcome-survey' );
  resumeSurveyStepKey.value = 'intent';
  currentView.value = 'home';
  showWelcomeSuccessSheet.value = true;
}

function reviseSurveyResponses() {
  showWelcomeSuccessSheet.value = false;
  currentSurveyStepIndex.value = 1;
  currentView.value = 'survey';
}

function completeCurrentTask( event ) {
  const index = currentTaskIndex.value;

  if ( index === -1 ) {
    return;
  }

  const task = progressionTasks.value[ index ];
  const isModifiedClick = Boolean(
    event?.metaKey || event?.ctrlKey || event?.shiftKey || event?.altKey
  );

  if ( isModifiedClick ) {
    markTaskCompleted( task.key );

    if ( task.action === 'enrich' ) {
      editsCount.value += 4;
      thanksCount.value += 2;
      streakCount.value = Math.max( streakCount.value, 3 );
    }

    if ( task.key === 'first-edit' ) {
      editsCount.value += 1;
      streakCount.value = Math.max( streakCount.value, 1 );
    }

    return;
  }

  if ( task.action === 'survey' ) {
    currentSurveyStepIndex.value = Math.max( 1, surveySteps.value.findIndex( ( step ) => step === resumeSurveyStepKey.value ) );
    currentView.value = 'survey';
    return;
  }

  if ( task.action === 'quiz' ) {
    currentQuizIndex.value = 0;
    currentView.value = 'quiz';
    return;
  }

  if ( task.action === 'video' ) {
    videoCompleted.value = false;
    showVideoBackButton.value = false;
    currentView.value = 'video';
    return;
  }

  if ( task.action === 'edit' ) {
    goToSuggestedEdits();
    return;
  }

  markTaskCompleted( task.key );
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
  showWelcomeSuccessSheet.value = false;
  currentView.value = 'home';
}

function goToArticle() {
  currentView.value = 'article';
}

function setActiveTask( taskKey ) {
  const task = progressionTasks.value.find( ( item ) => item.key === taskKey );

  if ( !task || task.completed ) {
    return;
  }

  activeTaskKey.value = taskKey;
}

function goToSuggestedEdits() {
  currentSuggestedEditIndex.value = 0;
  currentView.value = 'suggested-edits';
  requestAnimationFrame( () => {
    if ( suggestedEditsViewport.value ) {
      scrollToRenderedSuggestion( remainingSuggestedEdits.value.length > 1 ? 1 : 0 );
    }
  } );
}

function goToWikiMinuteLibrary() {
  currentView.value = 'wiki-minute-library';
}

function openWikiMinuteModal( video ) {
  selectedWikiMinuteVideo.value = video;
  showWikiMinuteModal.value = true;
}

function closeWikiMinuteModal() {
  showWikiMinuteModal.value = false;
  selectedWikiMinuteVideo.value = null;
}

function handleWikiMinuteEnded() {
  videoCompleted.value = true;
  showVideoBackButton.value = true;
  markTaskCompleted( 'wiki-minute' );
}

function handleWikiMinuteTimeUpdate( event ) {
  const player = event.target;

  if ( !player?.duration || Number.isNaN( player.duration ) ) {
    return;
  }

  if ( player.duration - player.currentTime <= 8 ) {
    showVideoBackButton.value = true;
  }
}

function completeVideoAndReturnHome() {
  if ( !progressionTasks.value.find( ( task ) => task.key === 'wiki-minute' )?.completed ) {
    markTaskCompleted( 'wiki-minute' );
  }

  videoCompleted.value = true;
  showVideoBackButton.value = true;
  currentView.value = 'home';
}

function resolveSuggestedEdit( mode, suggestionKey ) {
  const suggestion = currentSuggestedEdit.value;

  if ( !suggestion || suggestion.key !== suggestionKey ) {
    return;
  }

  suggestion.resolving = mode;
  if ( mode === 'complete' && suggestion.resolvedSnippetHtml ) {
    suggestion.articleSnippetHtml = suggestion.resolvedSnippetHtml;
  }

  if ( mode === 'complete' ) {
    editsCount.value += 1;
    streakCount.value = Math.max( streakCount.value, 1 );

    if ( !progressionTasks.value.find( ( task ) => task.key === 'first-edit' )?.completed ) {
      markTaskCompleted( 'first-edit' );
    }
  }

  setTimeout( () => {
    suggestion.resolved = true;
    suggestion.resolving = '';

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

function handleSuggestedEditsScroll( event ) {
  const viewport = event.target;
  const firstCard = viewport.querySelector( '.suggested-edit-card' );

  if ( !firstCard ) {
    return;
  }

  const styles = window.getComputedStyle( viewport.querySelector( '.suggested-edits-page__track' ) );
  const gap = Number.parseFloat( styles.columnGap || styles.gap || '0' );
  const cardWidth = firstCard.getBoundingClientRect().width + gap;
  const rawIndex = Math.round( viewport.scrollLeft / cardWidth );
  const itemCount = remainingSuggestedEdits.value.length;

  if ( itemCount <= 1 ) {
    currentSuggestedEditIndex.value = 0;
    return;
  }

  if ( rawIndex <= 0 ) {
    currentSuggestedEditIndex.value = itemCount - 1;
    requestAnimationFrame( () => {
      scrollToRenderedSuggestion( itemCount );
    } );
    return;
  }

  if ( rawIndex >= itemCount + 1 ) {
    currentSuggestedEditIndex.value = 0;
    requestAnimationFrame( () => {
      scrollToRenderedSuggestion( 1 );
    } );
    return;
  }

  currentSuggestedEditIndex.value = Math.max( 0, Math.min( itemCount - 1, rawIndex - 1 ) );
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

  markTaskCompleted( 'five-pillars' );
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

function markTaskCompleted( taskKey ) {
  progressionTasks.value = progressionTasks.value.map( ( task ) =>
    task.key === taskKey ? { ...task, completed: true } : task
  );
  recentlyCompletedTaskKey.value = taskKey;
  setTimeout( () => {
    if ( recentlyCompletedTaskKey.value === taskKey ) {
      recentlyCompletedTaskKey.value = '';
    }
  }, 550 );
  maybePromoteToContributorPath();
}

function maybePromoteToContributorPath() {
  if ( hasContributorPath.value ) {
    return;
  }

  if ( progressionTasks.value.length > 0 && progressionTasks.value.every( ( task ) => task.completed ) ) {
    hasContributorPath.value = true;
    progressionTasks.value = createContributorTasks();
    recentlyCompletedTaskKey.value = '';
    showContributorDialog.value = true;
    activeTaskKey.value = progressionTasks.value[ 0 ]?.key || '';
  }
}

function scrollToRenderedSuggestion( renderedIndex ) {
  const viewport = suggestedEditsViewport.value;
  const firstCard = viewport?.querySelector( '.suggested-edit-card' );

  if ( !viewport || !firstCard ) {
    return;
  }

  const styles = window.getComputedStyle( viewport.querySelector( '.suggested-edits-page__track' ) );
  const gap = Number.parseFloat( styles.columnGap || styles.gap || '0' );
  const cardWidth = firstCard.getBoundingClientRect().width + gap;
  viewport.scrollLeft = renderedIndex * cardWidth;
}

function dismissContributorSheet() {
  showContributorDialog.value = false;
  triggerProfileBadgeGlow();
}

function triggerProfileBadgeGlow() {
  showContributorBadgeGlow.value = true;
  setTimeout( () => {
    showContributorBadgeGlow.value = false;
  }, 1400 );
}
</script>

<template>
  <ChromeWrapper
    :show-footer="shouldShowChromeFooter"
    :show-site-header="shouldShowChromeSiteHeader"
    :header-mode="chromeHeaderMode"
    @go-home="goToHomepage"
    @go-article="goToArticle"
    @log-out="currentView = 'signup'"
  >
    <section v-if="currentView === 'signup'" class="account-creation-page">
      <div class="account-creation-page__panel">
        <div class="account-creation-page__copy">
          <h1 class="account-creation-page__title">
            Create your account
          </h1>
        </div>

        <div class="account-creation-form">
          <CdxField class="account-creation-form__field">
            <template #label>
              Username
            </template>
            <CdxTextInput
              v-model="accountUsername"
              aria-label="Username"
            />
          </CdxField>

          <CdxField class="account-creation-form__field">
            <template #label>
              Password
            </template>
            <CdxTextInput
              v-model="accountPassword"
              input-type="password"
              aria-label="Password"
            />
          </CdxField>

          <CdxField class="account-creation-form__field">
            <template #label>
              Confirm password
            </template>
            <CdxTextInput
              v-model="accountConfirmPassword"
              input-type="password"
              aria-label="Confirm password"
            />
          </CdxField>

          <CdxField class="account-creation-form__field">
            <template #label>
              Email address
            </template>
            <CdxTextInput
              input-type="email"
              placeholder="Enter your email address"
              aria-label="Email address"
            />
          </CdxField>
        </div>

        <p class="account-creation-page__disclaimer">
          This site is protected by hCaptcha and its
          <a href="https://www.hcaptcha.com/privacy" target="_blank" rel="noreferrer">Privacy Policy</a>
          and
          <a href="https://www.hcaptcha.com/terms" target="_blank" rel="noreferrer">Terms of Service</a>
          apply.
        </p>

        <CdxButton
          class="account-creation-page__submit"
          action="progressive"
          weight="primary"
          size="large"
          @click="goToWelcomeSurvey"
        >
          Create your account
        </CdxButton>
      </div>
    </section>

    <section
      v-else-if="currentView === 'survey'"
      class="survey-page"
      :class="{
        'survey-page--welcome': currentSurveyStep === 'welcome',
        'survey-page--steps': currentSurveyStep !== 'welcome'
      }"
    >
      <div v-if="currentSurveyStep === 'welcome'" class="survey-page__panel survey-page__panel--welcome">
        <div class="survey-page__hero-banner">
          <img
            class="survey-page__hero-image survey-page__hero-image--welcome"
            :src="welcomeSurveyHeroImage"
            alt="Animated Wikipedia welcome illustration"
          >
        </div>

        <div class="survey-page__copy">
          <h1 class="survey-page__title">
            Welcome, {{ userName }}
          </h1>
          <p class="survey-page__body survey-page__body--welcome">
            Wikipedia is built by people like you. Respond some quick questions to customize your
            experience.
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
            weight="quiet"
            size="large"
            @click="skipSurvey"
          >
            Not now
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
              <strong>{{ currentSurveyQuestionNumber }}</strong> of {{ totalSurveyQuestionCount }}
            </p>
          </div>
        </div>

        <template v-if="currentSurveyStep === 'intent'">
          <div class="survey-page__copy survey-page__copy--step">
            <h1 class="survey-page__title survey-page__title--question">
              What would you like to do on Wikipedia?
            </h1>
            <p class="survey-page__body">
              Choose what you want help with. We’ll set up your dashboard to match.
            </p>
          </div>

          <div class="survey-page__options">
            <button
              v-for="option in intentOptions"
              :key="option.key"
              class="survey-card"
              :class="{ 'survey-card--selected': selectedIntent === option.key }"
              type="button"
              @click="selectedIntent = option.key"
            >
              <span class="survey-card__marker" aria-hidden="true">
                <CdxIcon v-if="selectedIntent === option.key" :icon="cdxIconCheck" />
                <span v-else>{{ intentOptions.findIndex( ( item ) => item.key === option.key ) + 1 }}</span>
              </span>
              <span class="survey-card__content">
                <span class="survey-card__title">{{ option.title }}</span>
                <span v-if="option.description" class="survey-card__description">{{ option.description }}</span>
              </span>
            </button>
          </div>
        </template>

        <template v-else-if="currentSurveyStep === 'reason'">
          <div class="survey-page__copy survey-page__copy--step">
            <h1 class="survey-page__title survey-page__title--question">
              Have you edited Wikipedia before?
            </h1>
            <p class="survey-page__body">
              We’ll set up your path based on your experience.
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
              <span class="survey-card__marker" aria-hidden="true">
                <CdxIcon v-if="selectedReason === option.key" :icon="cdxIconCheck" />
                <span v-else>{{ reasonOptions.findIndex( ( item ) => item.key === option.key ) + 1 }}</span>
              </span>
              <span class="survey-card__content">
                <span class="survey-card__title">{{ option.title }}</span>
                <span v-if="option.description" class="survey-card__description">{{ option.description }}</span>
              </span>
            </button>
          </div>
        </template>

        <template v-else-if="currentSurveyStep === 'topics'">
          <div class="survey-page__copy survey-page__copy--step">
            <h1 class="survey-page__title survey-page__title--question">
              What topics you are interested in?
            </h1>
            <p class="survey-page__body">
              Choose at least 4 topics you are interested in editing.
            </p>
          </div>

          <div class="survey-topic-groups">
            <section
              v-for="category in topicCategories"
              :key="category.key"
              class="survey-topic-group"
            >
              <div class="survey-topic-group__header">
                <h2 class="survey-topic-group__title">
                  {{ category.title }}
                </h2>
                <CdxButton
                  class="survey-topic-group__select-all"
                  weight="quiet"
                  action="progressive"
                  @click="selectAllTopicsInCategory( category )"
                >
                  Select all
                </CdxButton>
              </div>

              <div class="survey-topic-group__chips">
                <button
                  v-for="topic in category.topics"
                  :key="topic"
                  class="survey-topic-chip"
                  :class="{ 'survey-topic-chip--selected': selectedTopics.includes( topic ) }"
                  type="button"
                  @click="toggleGroupedTopic( topic )"
                >
                  <span
                    v-if="selectedTopics.includes( topic )"
                    class="survey-topic-chip__icon"
                    aria-hidden="true"
                  >
                    <CdxIcon :icon="cdxIconCheck" />
                  </span>
                  <span>{{ topic }}</span>
                </button>
              </div>
            </section>
          </div>
        </template>

        <template v-else-if="currentSurveyStep === 'languages'">
          <div class="survey-page__copy survey-page__copy--step">
            <h1 class="survey-page__title survey-page__title--question">
              Which languages do you read and write in?
            </h1>
            <p class="survey-page__body">
              Wikipedia is available in nearly 300 languages. You can add up to 10 languages you read and write.
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

          <CdxCheckbox
            v-if="selectedLanguages.length > 1"
            v-model="isInterestedInTranslation"
            class="survey-page__translation-optin"
          >
            I am interested in translating Wikipedia articles
          </CdxCheckbox>
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
          <div class="survey-page__footer-nav-inner">
            <CdxButton
              v-if="currentSurveyStep !== 'intent'"
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
      </div>
    </section>

    <section v-else-if="currentView === 'video'" class="video-page">
      <header class="video-page__header">
        <CdxButton
          class="video-page__back"
          weight="quiet"
          aria-label="Go back"
          @click="goToHomepage"
        >
          <CdxIcon :icon="cdxIconArrowPrevious" />
        </CdxButton>
        <h1 class="video-page__header-title">
          Wiki Minute video
        </h1>
      </header>

      <div class="video-page__player-wrap">
        <video
          class="video-page__player"
          controls
          playsinline
          @timeupdate="handleWikiMinuteTimeUpdate"
          @ended="handleWikiMinuteEnded"
        >
          <source :src="wikiMinuteVideoSource" type="video/webm">
        </video>
      </div>

      <div v-if="showVideoBackButton" class="video-page__actions">
        <CdxButton
          class="video-page__button"
          action="default"
          weight="normal"
          @click="completeVideoAndReturnHome"
        >
          <template #icon>
            <CdxIcon :icon="cdxIconCheck" />
          </template>
          Completed video
        </CdxButton>
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
        <div class="quiz-page__footer-nav-inner">
          <CdxButton
            v-if="currentQuizIndex > 0"
            class="quiz-page__nav-link"
            weight="quiet"
            @click="goToPreviousQuizStep"
          >
            <CdxIcon :icon="cdxIconArrowPrevious" />
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
            <CdxIcon :icon="cdxIconArrowNext" />
          </CdxButton>
        </div>
      </div>
    </section>

    <section v-else-if="currentView === 'wiki-minute-library'" class="wiki-minute-library-page">
      <header class="wiki-minute-library-page__header">
        <CdxButton
          class="wiki-minute-library-page__back"
          weight="quiet"
          aria-label="Go back"
          @click="goToHomepage"
        >
          <CdxIcon :icon="cdxIconArrowPrevious" />
        </CdxButton>
        <h1 class="wiki-minute-library-page__header-title">
          Wiki Minute videos
        </h1>
      </header>

      <section class="wiki-minute-library-page__list">
        <button
          v-for="video in wikiMinuteVideos"
          :key="video.key"
          class="wiki-minute-library-item"
          type="button"
          @click="openWikiMinuteModal( video )"
        >
          <video
            class="wiki-minute-library-item__thumbnail"
            :src="video.source"
            muted
            playsinline
            preload="metadata"
          ></video>
          <div class="wiki-minute-library-item__copy">
            <span class="wiki-minute-library-item__title">{{ video.title }}</span>
            <span
              v-if="video.watched && hasCompletedWikiMinuteTask"
              class="wiki-minute-library-item__status"
            >
              <CdxIcon :icon="cdxIconCheck" />
              Watched
            </span>
          </div>
        </button>
      </section>
    </section>

    <template v-else-if="currentView === 'suggested-edits'">
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
        <span class="suggested-edits-page__filters" aria-hidden="true">
          <CdxIcon :icon="cdxIconConfigure" />
        </span>
      </header>

      <section class="suggested-edits-page">
        <p class="suggested-edits-page__count">
          <strong>{{ currentSuggestedEditSequenceNumber }}</strong> of {{ remainingSuggestedEdits.length }} suggestions
        </p>

        <div
          ref="suggestedEditsViewport"
          class="suggested-edits-page__viewport"
          @scroll="handleSuggestedEditsScroll"
        >
          <div
            class="suggested-edits-page__track"
          >
            <article
              v-for="item in renderedSuggestedEdits"
              :key="item.key"
              class="suggested-edit-card"
              :class="{
                'suggested-edit-card--active': item.suggestion.key === currentSuggestedEdit?.key,
                'suggested-edit-card--completing': item.suggestion.resolving === 'complete',
                'suggested-edit-card--dismissing': item.suggestion.resolving === 'dismiss'
              }"
            >
              <div class="suggested-edit-card__body">
                <img
                  class="suggested-edit-card__thumbnail"
                  :src="item.suggestion.articleImage"
                  :alt="item.suggestion.articleTitle"
                >
                <div class="suggested-edit-card__copy">
                  <h2 class="suggested-edit-card__title">
                    {{ item.suggestion.articleTitle }}
                  </h2>
                  <p class="suggested-edit-card__description">
                    {{ item.suggestion.articleDescription }}
                  </p>
                </div>

                <div
                  class="suggested-edit-card__excerpt"
                  v-html="item.suggestion.articleSnippetHtml"
                ></div>
              </div>

              <div
                v-if="item.suggestion.resolving === 'complete'"
                class="suggested-edit-card__panel suggested-edit-card__panel--success"
              >
                <div class="suggested-edit-card__success-row">
                  <span class="suggested-edit-card__success-icon">
                    <CdxIcon :icon="cdxIconCheck" />
                  </span>
                  <strong>{{ item.suggestion.successTitle }}</strong>
                </div>
                <p class="suggested-edit-card__panel-description">
                  {{ item.suggestion.successMessage }}
                </p>
              </div>

              <div
                v-else
                class="suggested-edit-card__panel suggested-edit-card__panel--task"
              >
                <h3 class="suggested-edit-card__panel-title">
                  {{ item.suggestion.taskTitle }}
                </h3>
                <p class="suggested-edit-card__panel-description">
                  {{ item.suggestion.taskDescription }}
                </p>
                <p
                  v-if="item.suggestion.template === 'englishSpelling' && item.suggestion.question"
                  class="suggested-edit-card__question"
                >
                  {{ item.suggestion.question }}
                </p>

                <div class="suggested-edit-card__actions">
                  <CdxButton @click="resolveSuggestedEdit( 'complete', item.suggestion.key )">
                    {{ item.suggestion.primaryLabel }}
                  </CdxButton>
                  <CdxButton @click="resolveSuggestedEdit( 'dismiss', item.suggestion.key )">
                    {{ item.suggestion.secondaryLabel }}
                  </CdxButton>
                </div>
              </div>
            </article>
          </div>
        </div>
      </section>
    </template>

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
          <div class="homepage__meta">
            <span
              class="homepage__level-chip"
              :class="{ 'homepage__level-chip--glow': showContributorBadgeGlow }"
            >
              <CdxInfoChip :status="profileLevelStatus">
                {{ profileLevelLabel }}
              </CdxInfoChip>
            </span>
            <span>{{ profileJoinedText }}</span>
          </div>
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
            @click="setActiveTask( task.key )"
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

              <CdxButton
                v-if="index === currentTaskIndex"
                class="progression-item__button"
                action="progressive"
                weight="primary"
                size="small"
                @click.stop="completeCurrentTask"
              >
                <template #icon-end>
                  <CdxIcon :icon="cdxIconArrowNext" />
                </template>
                {{ task.buttonLabel }}
              </CdxButton>
            </div>
          </article>
        </div>

      </section>

      <section
        v-if="hasCompletedFirstEditTask"
        class="module module--suggested-edits"
      >
        <div class="module__suggested-edits-header">
          <div>
            <h2 class="module__title module__title--small">
              Suggested edits
            </h2>
            <p class="module__body module__body--suggested-edits">
              1 of 324 suggestions
            </p>
          </div>
          <CdxButton
            class="module__suggested-edits-link"
            weight="quiet"
            action="progressive"
            @click="goToSuggestedEdits"
          >
            View all
          </CdxButton>
        </div>

        <div class="module__suggested-edits-carousel">
          <CdxCard
            v-for="suggestion in homepageSuggestedEditsPreview"
            :key="suggestion.key"
            class="module__suggested-edits-card"
            :thumbnail="{ url: suggestion.articleImage }"
          >
            <template #title>
              {{ suggestion.articleTitle }}
            </template>
            <template #description>
              {{ suggestion.articleDescription }}
            </template>
          </CdxCard>
        </div>
      </section>

      <section
        v-if="hasCompletedWikiMinuteTask"
        class="module module--wiki-minute"
      >
        <div class="module__wiki-minute-header">
          <div>
            <h2 class="module__title module__title--small">
              Wiki Minute videos
            </h2>
            <p class="module__body module__body--wiki-minute">
              1 min of knowledge
            </p>
          </div>
          <CdxButton
            class="module__wiki-minute-link"
            weight="quiet"
            action="progressive"
            @click="goToWikiMinuteLibrary"
          >
            View all
          </CdxButton>
        </div>

        <button
          class="module__wiki-minute-feature"
          type="button"
          @click="openWikiMinuteModal( featuredWikiMinuteVideo )"
        >
          <video
            class="module__wiki-minute-thumbnail"
            :src="featuredWikiMinuteVideo.source"
            muted
            playsinline
            preload="metadata"
          ></video>
          <span class="module__wiki-minute-title">{{ featuredWikiMinuteVideo.title }}</span>
        </button>
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
            <CdxButton class="module__mentor-button" size="small">
              Ask your mentor
            </CdxButton>
          </div>
        </div>
      </section>

      <section class="homepage__impact-grid">
        <article class="impact-card">
          <strong class="impact-card__value">
            <CdxIcon :icon="cdxIconEdit" />
            <span>{{ editsCount }}/5</span>
          </strong>
          <span class="impact-card__label">edits</span>
        </article>
        <article class="impact-card">
          <strong class="impact-card__value">
            <CdxIcon :icon="cdxIconHeart" />
            <span>{{ thanksCount }}</span>
          </strong>
          <span class="impact-card__label">thanks</span>
        </article>
        <article class="impact-card">
          <strong class="impact-card__value">
            <CdxIcon :icon="cdxIconCalendar" />
            <span>{{ streakCount }}</span>
          </strong>
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

    <transition name="contributor-sheet-fade">
      <div
        v-if="showWelcomeSuccessSheet"
        class="contributor-sheet"
        role="dialog"
        aria-modal="true"
        aria-labelledby="welcome-success-sheet-title"
      >
        <button
          class="contributor-sheet__overlay contributor-sheet__overlay--white"
          type="button"
          aria-label="Close welcome success message"
          @click="showWelcomeSuccessSheet = false"
        ></button>

        <div class="contributor-sheet__panel">
          <div class="contributor-sheet__content">
            <img
              class="contributor-sheet__image"
              :src="contributorDialogImage"
              alt="Wikipedia contributor achievement illustration"
            >
            <h2 id="welcome-success-sheet-title" class="contributor-sheet__title">
              You're all set, {{ userName }}
            </h2>
            <p class="contributor-sheet__body">
              Your homepage is ready. We've picked your first edit based on your interests.
            </p>
          </div>

          <div class="contributor-sheet__actions contributor-sheet__actions--stacked">
            <CdxButton
              class="contributor-sheet__button"
              action="progressive"
              weight="primary"
              size="medium"
              @click="goToHomepage"
            >
              Go to homepage
            </CdxButton>
            <CdxButton
              class="contributor-sheet__button contributor-sheet__button--quiet"
              weight="quiet"
              size="medium"
              @click="reviseSurveyResponses"
            >
              Revise responses
            </CdxButton>
          </div>
        </div>
      </div>
    </transition>

    <transition name="contributor-sheet-fade">
      <div
        v-if="showContributorDialog"
        class="contributor-sheet"
        role="dialog"
        aria-modal="true"
        aria-labelledby="contributor-sheet-title"
      >
        <button
          class="contributor-sheet__overlay"
          type="button"
          aria-label="Close contributor message"
          @click="dismissContributorSheet"
        ></button>

        <div class="contributor-sheet__panel">
          <div class="contributor-sheet__content">
            <img
              class="contributor-sheet__image"
              :src="contributorDialogImage"
              alt="Wikipedia contributor achievement illustration"
            >
            <h2 id="contributor-sheet-title" class="contributor-sheet__title">
              You've completed the Beginner path
            </h2>
            <p class="contributor-sheet__body">
              Congratulations! You're now a Contributor. A new path is ready for you to keep
              improving your editor skills.
            </p>
          </div>

          <div class="contributor-sheet__actions">
            <CdxButton
              class="contributor-sheet__button"
              action="progressive"
              weight="primary"
              size="medium"
              @click="dismissContributorSheet"
            >
              Got it
            </CdxButton>
          </div>
        </div>
      </div>
    </transition>

    <transition name="contributor-sheet-fade">
      <div
        v-if="showWikiMinuteModal && selectedWikiMinuteVideo"
        class="wiki-minute-modal"
        role="dialog"
        aria-modal="true"
        aria-labelledby="wiki-minute-modal-title"
      >
        <button
          class="wiki-minute-modal__overlay"
          type="button"
          aria-label="Close video modal"
          @click="closeWikiMinuteModal"
        ></button>

        <div class="wiki-minute-modal__panel">
          <header class="wiki-minute-modal__header">
            <CdxButton
              class="wiki-minute-modal__back"
              weight="quiet"
              aria-label="Close video modal"
              @click="closeWikiMinuteModal"
            >
              <CdxIcon :icon="cdxIconArrowPrevious" />
            </CdxButton>
            <h1 id="wiki-minute-modal-title" class="wiki-minute-modal__title">
              {{ selectedWikiMinuteVideo.title }}
            </h1>
          </header>

          <video
            class="wiki-minute-modal__player"
            :src="selectedWikiMinuteVideo.source"
            controls
            playsinline
            autoplay
          ></video>
        </div>
      </div>
    </transition>
  </ChromeWrapper>
</template>

<style scoped>
.survey-page,
.account-creation-page,
.homepage,
.quiz-page,
.suggested-edits-page,
.article-page {
  min-height: 100%;
  background-color: var(--background-color-base);
}

.survey-page--steps {
  background-color: var(--background-color-neutral-subtle);
}

.account-creation-page__panel,
.survey-page__panel,
.homepage,
.quiz-page,
.suggested-edits-page,
.article-page {
  max-width: 30rem;
  padding: var(--spacing-150) 16px;
}

.account-creation-page__panel {
  display: flex;
  min-height: calc(100vh - 54px);
  flex-direction: column;
  justify-content: flex-start;
  padding-top: 16px;
  background-color: var(--background-color-base);
}

.account-creation-page__copy {
  display: flex;
  flex-direction: column;
  gap: 0;
}

.account-creation-page__title {
  margin: 0;
  color: var(--color-emphasized);
  font-family: var(--font-family-system-sans);
  font-size: 24px;
  line-height: 32px;
  font-weight: var(--font-weight-bold);
}

.account-creation-form {
  display: flex;
  flex-direction: column;
  gap: 24px;
  margin-top: 24px;
}

.account-creation-form__field {
  margin: 0;
}

.account-creation-form__field :deep(.cdx-text-input__input) {
  min-height: 44px;
  padding-right: 12px;
  padding-left: 12px;
}

.account-creation-page__disclaimer {
  margin: 0;
  margin-top: 24px;
  color: var(--color-subtle);
  font-size: 14px;
  line-height: 20px;
}

.account-creation-page__disclaimer a {
  color: var(--color-progressive);
  text-decoration: none;
}

.account-creation-page__disclaimer a:hover {
  text-decoration: underline;
}

.account-creation-page__submit {
  width: 100%;
  margin-top: 16px;
  justify-content: center;
}

.survey-page__panel--welcome,
.survey-page__panel--step {
  display: flex;
  flex-direction: column;
}

.survey-page__panel--welcome {
  gap: var(--spacing-300);
  padding-top: 0;
  padding-bottom: 180px;
}

.survey-page__hero-banner {
  display: flex;
  align-items: center;
  justify-content: center;
  width: calc(100% + 32px);
  aspect-ratio: 16 / 9;
  margin-inline: -16px;
  background-color: #00a0ff;
  overflow: hidden;
}

.survey-page__hero-image {
  display: block;
  width: calc(100% + 32px);
  height: auto;
  min-height: 180px;
  margin-inline: -16px;
  margin-top: 0;
  border: 0;
  border-radius: 0;
  background-color: transparent;
  object-fit: cover;
}

.survey-page__hero-image--welcome {
  width: 180px;
  height: 180px;
  min-height: 0;
  margin-inline: 0;
  object-fit: contain;
}

.survey-page__hero-image--success {
  filter: hue-rotate(42deg) saturate(1.15);
}

.survey-page__panel--welcome .survey-page__copy {
  margin-top: -8px;
}

.survey-page__panel--step {
  min-height: calc(100vh - 54px);
  gap: 24px;
  padding-bottom: 96px;
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
.module__body,
.progression-item__description {
  margin: 0;
  color: var(--color-emphasized);
  font-size: 1rem;
  line-height: 1.5;
}

.survey-page__body--welcome {
  font-size: 16px;
  line-height: 22px;
}

.survey-page__title--success {
  font-family: var(--font-family-system-sans);
  font-size: 28px;
  line-height: 38px;
  font-weight: var(--font-weight-bold);
}

.homepage__meta {
  display: flex;
  align-items: center;
  gap: 8px;
  color: var(--color-subtle);
  font-size: 14px;
  line-height: 20px;
}

.homepage__level-chip {
  position: relative;
  display: inline-flex;
  border-radius: 999px;
}

.homepage__level-chip:deep(.cdx-info-chip) {
  position: relative;
}

.homepage__level-chip--glow {
  animation: homepage-level-chip-glow 1100ms ease;
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
  height: 8px;
}

.survey-page__progress-bar-wrap :deep(.cdx-progress-bar) {
  width: 100%;
  height: 8px;
}

.survey-page__progress-bar-wrap :deep(.cdx-progress-bar__bar),
.quiz-page__progress-block :deep(.cdx-progress-bar__bar) {
  overflow: hidden;
  border: 0;
  box-shadow: none;
  background-image: none;
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
  position: fixed;
  right: 0;
  bottom: calc(48px + env(safe-area-inset-bottom, 0px));
  left: 0;
  z-index: 1;
  max-width: 30rem;
  margin: 0 auto;
  padding: 0 16px;
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

.survey-topic-groups {
  display: flex;
  flex-direction: column;
  gap: 32px;
}

.survey-topic-group {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.survey-topic-group__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}

.survey-topic-group__title {
  margin: 0;
  color: var(--color-emphasized);
  font-size: 18px;
  line-height: 30px;
  font-weight: var(--font-weight-bold);
}

.survey-topic-group__select-all:deep(.cdx-button) {
  font-size: 14px;
}

.survey-topic-group__chips {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.survey-topic-chip {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  min-height: 44px;
  border: 1px solid var(--border-color-base);
  border-radius: 999px;
  background-color: var(--background-color-base);
  padding: 0 16px;
  color: var(--color-base);
  font-size: 16px;
  line-height: 22px;
}

.survey-topic-chip--selected {
  border-color: var(--border-color-progressive);
  background-color: var(--background-color-progressive-subtle);
  color: var(--color-progressive);
}

.survey-topic-chip__icon {
  display: inline-flex;
  align-items: center;
  color: var(--color-progressive);
}

.survey-topic-chip__icon :deep(.cdx-icon) {
  color: var(--color-progressive);
}

.survey-card {
  display: flex;
  align-items: center;
  gap: 16px;
  width: 100%;
  border: var(--border-base);
  border-radius: var(--border-radius-base);
  background-color: var(--background-color-base);
  padding: 12px;
  text-align: left;
  border-radius: 999px;
}

.survey-card--selected {
  border-color: var(--border-color-progressive);
  background-color: var(--background-color-progressive-subtle);
}

.survey-card__marker {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  flex-shrink: 0;
  border: 1px solid var(--border-color-base);
  border-radius: 50%;
  background-color: var(--background-color-base);
  color: var(--color-base);
  font-size: 20px;
  line-height: 1;
}

.survey-card--selected .survey-card__marker {
  border-color: var(--background-color-progressive);
  background-color: var(--background-color-progressive);
  color: var(--color-inverted);
}

.survey-card--selected .survey-card__marker :deep(.cdx-icon) {
  color: var(--color-inverted);
}

.survey-card__marker :deep(.cdx-icon) {
  width: 20px;
  height: 20px;
}

.survey-card__content {
  display: flex;
  min-width: 0;
  flex: 1 1 auto;
  flex-direction: column;
  gap: 4px;
}

.progression-item__title,
.module__title,
.homepage__tab {
  color: var(--color-subtle);
  font-size: 1rem;
  font-weight: var(--font-weight-bold);
  line-height: 1.4;
}

.survey-card__title {
  color: var(--color-emphasized);
  font-size: 1.125rem;
  font-weight: var(--font-weight-regular);
  line-height: 1.5rem;
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
  border: 1px solid var(--border-color-base);
  border-radius: 999px;
  background-color: transparent;
  padding: 0.625rem 0.875rem;
  color: var(--color-base);
  font-size: 1rem;
  font-weight: var(--font-weight-regular);
}

.survey-chip--add {
  border-color: var(--border-color-interactive);
  background-color: transparent;
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

.survey-page__translation-optin {
  margin-top: 24px;
}

.survey-page__field-label {
  color: var(--color-emphasized);
  font-size: 1rem;
  line-height: 1.4;
}

.survey-page__footer-nav {
  position: fixed;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 1;
  padding-top: 12px;
  padding-bottom: calc(12px + env(safe-area-inset-bottom, 0px));
  background-color: var(--background-color-neutral-subtle);
}

.survey-page__footer-nav-inner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  max-width: 30rem;
  margin: 0 auto;
  padding: 0 16px;
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

.progression-item:not(.progression-item--complete) {
  cursor: pointer;
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
  width: 24px;
  height: 24px;
  border: 1px solid var(--border-color-interactive);
  border-radius: 50%;
  background-color: var(--background-color-base);
  color: var(--color-base);
  font-size: 16px;
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
  background-color: var(--border-color-interactive);
}

.progression-item--complete .progression-item__marker {
  border-color: var(--color-icon-success);
  background-color: var(--color-icon-success);
  color: var(--color-inverted);
}

.progression-item--complete .progression-item__line {
  background-color: var(--border-color-success);
}

.progression-item--complete .progression-item__title {
  color: var(--color-placeholder);
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
  padding-bottom: 16px;
}

.progression-item:last-child .progression-item__content {
  padding-bottom: 0;
}

.progression-item__heading {
  display: flex;
  align-items: center;
  gap: var(--spacing-50);
  min-height: 24px;
}

.progression-item__status {
  display: none;
}

.progression-item__button {
  align-self: flex-start;
}

.progression-item__button:deep(.cdx-button) {
  font-size: 14px;
}

.module__mentor {
  display: flex;
  gap: var(--spacing-100);
}

.module--mentor {
  border-color: var(--border-color-muted);
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
  color: var(--color-subtle);
  font-size: 14px;
  line-height: 20px;
}

.module__mentor-button {
  align-self: flex-start;
  margin-top: 8px;
}

.module__mentor-button:deep(.cdx-button) {
  min-height: 32px;
}

.module--suggested-edits {
  display: flex;
  flex-direction: column;
  gap: 12px;
  background-color: var(--background-color-progressive-subtle);
}

.module__suggested-edits-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 12px;
}

.module__body--suggested-edits {
  color: var(--color-subtle);
  font-size: 14px;
  line-height: 20px;
}

.module__suggested-edits-link {
  flex-shrink: 0;
}

.module__suggested-edits-link:deep(.cdx-button) {
  font-size: 14px;
  font-weight: var(--font-weight-bold);
}

.module__suggested-edits-carousel {
  display: flex;
  gap: 12px;
  overflow-x: auto;
  padding-bottom: 2px;
}

.module__suggested-edits-card {
  min-width: 17rem;
  flex: 0 0 17rem;
}

.module__suggested-edits-card:deep(.cdx-card) {
  border-radius: var(--border-radius-base);
}

.module__suggested-edits-card:deep(.cdx-card__text__title) {
  color: var(--color-base);
  font-size: 14px;
  line-height: 20px;
  font-weight: var(--font-weight-bold);
}

.module__suggested-edits-card:deep(.cdx-card__text__description) {
  color: var(--color-subtle);
  font-size: 14px;
  line-height: 20px;
}

.module--wiki-minute {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.module__wiki-minute-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 12px;
}

.module__body--wiki-minute {
  color: var(--color-subtle);
  font-size: 14px;
  line-height: 20px;
}

.module__wiki-minute-link {
  flex-shrink: 0;
}

.module__wiki-minute-link:deep(.cdx-button) {
  font-size: 14px;
  font-weight: var(--font-weight-bold);
}

.module__wiki-minute-feature {
  display: grid;
  grid-template-columns: 7rem 1fr;
  gap: 12px;
  align-items: center;
  border: 0;
  background: transparent;
  padding: 0;
  text-align: left;
}

.module__wiki-minute-thumbnail {
  display: block;
  width: 100%;
  aspect-ratio: 16 / 9;
  border: 1px solid var(--border-color-subtle);
  border-radius: var(--border-radius-base);
  object-fit: cover;
  background-color: #000;
}

.module__wiki-minute-title {
  color: var(--color-base);
  font-size: 16px;
  line-height: 22px;
}

.module--mentor .module__title,
.module--wiki-minute .module__title {
  color: var(--color-base);
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
  display: inline-flex;
  align-items: center;
  gap: 8px;
  color: var(--color-emphasized);
  font-size: 16px;
  line-height: 1;
}

.impact-card__value :deep(.cdx-icon) {
  width: 16px;
  height: 16px;
  color: var(--color-emphasized);
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

.video-page {
  display: flex;
  min-height: calc(100vh - 54px);
  flex-direction: column;
  background-color: var(--background-color-base);
}

.video-page__header {
  display: flex;
  align-items: center;
  gap: 4px;
  min-height: 48px;
  padding: 0 16px;
  border-bottom: 1px solid var(--border-color-subtle);
  background-color: var(--background-color-base);
}

.video-page__back {
  flex-shrink: 0;
}

.video-page__header-title {
  margin: 0;
  color: var(--color-emphasized);
  font-family: var(--font-family-system-sans);
  font-size: 18px;
  line-height: 22px;
  font-weight: var(--font-weight-bold);
}

.video-page__player-wrap {
  display: flex;
  width: calc(100% + 32px);
  margin-inline: -16px;
  background-color: var(--background-color-base);
}

.video-page__player {
  display: block;
  width: 100%;
  height: auto;
  background-color: #000;
}

.video-page__actions {
  padding: 24px 16px 48px;
  background-color: var(--background-color-base);
}

.video-page__button {
  width: 100%;
  justify-content: center;
}

.wiki-minute-library-page {
  display: flex;
  min-height: calc(100vh - 54px);
  flex-direction: column;
  background-color: var(--background-color-base);
}

.wiki-minute-library-page__header,
.wiki-minute-modal__header {
  display: flex;
  align-items: center;
  gap: 4px;
  min-height: 48px;
  padding: 0 16px;
  border-bottom: 1px solid var(--border-color-subtle);
  background-color: var(--background-color-base);
}

.wiki-minute-library-page__header-title,
.wiki-minute-modal__title {
  margin: 0;
  color: var(--color-emphasized);
  font-family: var(--font-family-system-sans);
  font-size: 18px;
  line-height: 22px;
  font-weight: var(--font-weight-bold);
}

.wiki-minute-library-page__list {
  display: flex;
  flex-direction: column;
  background-color: var(--background-color-base);
}

.wiki-minute-library-item {
  display: grid;
  grid-template-columns: 6rem 1fr;
  gap: 12px;
  align-items: center;
  border: 0;
  border-bottom: 1px solid var(--border-color-subtle);
  background-color: var(--background-color-base);
  padding: 12px 16px;
  text-align: left;
}

.wiki-minute-library-item__thumbnail {
  display: block;
  width: 100%;
  aspect-ratio: 16 / 9;
  border: 1px solid var(--border-color-subtle);
  border-radius: var(--border-radius-base);
  object-fit: cover;
  background-color: #000;
}

.wiki-minute-library-item__copy {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.wiki-minute-library-item__title {
  color: var(--color-base);
  font-size: 16px;
  line-height: 22px;
}

.wiki-minute-library-item__status {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  color: var(--color-progressive);
  font-size: 14px;
  line-height: 20px;
}

.wiki-minute-library-item__status :deep(.cdx-icon) {
  width: 14px;
  height: 14px;
}

.wiki-minute-modal {
  position: fixed;
  inset: 0;
  z-index: 25;
  display: flex;
  align-items: flex-end;
  justify-content: center;
}

.wiki-minute-modal__overlay {
  position: absolute;
  inset: 0;
  border: 0;
  background: rgb(0 0 0 / 45%);
}

.wiki-minute-modal__panel {
  position: relative;
  z-index: 1;
  display: flex;
  width: min(100%, 32rem);
  max-height: 100%;
  flex-direction: column;
  overflow: hidden;
  border-radius: 12px 12px 0 0;
  background-color: var(--background-color-base);
}

.wiki-minute-modal__player {
  display: block;
  width: 100%;
  height: auto;
  background-color: #000;
}

.contributor-sheet {
  position: fixed;
  inset: 0;
  z-index: 20;
  display: flex;
  align-items: flex-end;
  justify-content: center;
}

.contributor-sheet__overlay {
  position: absolute;
  inset: 0;
  border: 0;
  background-color: var(--background-color-backdrop-light);
}

.contributor-sheet__overlay--white {
  background-color: rgb(255 255 255 / 82%);
}

.contributor-sheet__panel {
  position: relative;
  z-index: 1;
  width: min(100%, 30rem);
  max-width: 30rem;
  border: var(--border-base);
  border-color: var(--border-color-muted);
  border-radius: 12px 12px 0 0;
  background-color: var(--background-color-base);
  box-shadow: 0 -4px 24px rgb(0 0 0 / 12%);
}

.contributor-sheet__content {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  padding: 24px 16px 0;
}

.contributor-sheet__image {
  display: block;
  width: 180px;
  max-width: 100%;
  height: auto;
  margin-bottom: 24px;
}

.contributor-sheet__title {
  margin: 0;
  color: var(--color-emphasized);
  font-family: var(--font-family-system-sans);
  font-size: 18px;
  line-height: 28px;
  font-weight: var(--font-weight-bold);
}

.contributor-sheet__body {
  margin: 8px 0 0;
  color: var(--color-base);
  font-size: 16px;
  line-height: 22px;
}

.contributor-sheet__actions {
  padding: 16px 16px 16px;
}

.contributor-sheet__actions--stacked {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.contributor-sheet__button,
.contributor-sheet__button:deep(.cdx-button) {
  width: 100%;
  justify-content: center;
}

.contributor-sheet__button--quiet:deep(.cdx-button) {
  border-color: transparent;
}

.quiz-page {
  display: flex;
  flex-direction: column;
  gap: 16px;
  padding-bottom: 96px;
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
  height: 8px;
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
  font-size: 18px;
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
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.quiz-option__button {
  display: flex;
  align-items: center;
  gap: 12px;
  width: 100%;
  border: var(--border-base);
  border-color: var(--border-color-base);
  border-radius: 999px;
  background: var(--background-color-base);
  padding: 12px;
  text-align: left;
}

.quiz-option--success .quiz-option__button {
  border-color: var(--border-color-success);
  background-color: var(--background-color-success-subtle);
}

.quiz-option--error .quiz-option__button {
  border-color: var(--border-color-error);
  background-color: var(--background-color-error-subtle);
}

.quiz-option__key {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  flex-shrink: 0;
  border: 1px solid var(--border-color-base);
  border-radius: 50%;
  color: var(--color-subtle);
  background-color: var(--background-color-base);
  font-size: 20px;
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
  font-weight: var(--font-weight-regular);
}

.quiz-option__message {
  margin: 0;
  padding: 0 12px;
  font-size: 14px;
  line-height: 22px;
}

.quiz-option__message--error {
  color: var(--color-error);
}

.quiz-option--success .quiz-option__text {
  color: var(--color-success);
}

.quiz-option--error .quiz-option__text {
  color: var(--color-error);
}

.quiz-page__footer-nav {
  position: fixed;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 1;
  padding-top: 12px;
  padding-bottom: calc(12px + env(safe-area-inset-bottom, 0px));
  background-color: var(--background-color-base);
}

.quiz-page__footer-nav-inner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  max-width: 30rem;
  margin: 0 auto;
  padding: 0 16px;
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
  min-height: calc(100vh - 48px);
  background-color: var(--background-color-neutral-subtle);
}

.suggested-edits-page__header {
  display: flex;
  align-items: center;
  gap: 4px;
  width: 100%;
  min-height: 48px;
  padding-right: 16px;
  padding-left: 16px;
  padding-top: 0;
  padding-bottom: 0;
  border-bottom: 1px solid var(--border-color-subtle);
  background-color: var(--background-color-base);
}

.suggested-edits-page__filters {
  margin-inline-start: auto;
  color: var(--color-base);
  font-size: 24px;
  line-height: 1;
  display: inline-flex;
  align-items: center;
}

.suggested-edits-page__header-title {
  margin: 0;
  color: var(--color-emphasized);
  font-family: var(--font-family-system-sans);
  font-size: 18px;
  line-height: 22px;
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
  flex: 1 1 auto;
  min-height: 0;
  overflow-x: auto;
  overflow-y: hidden;
  scroll-snap-type: x mandatory;
  -webkit-overflow-scrolling: touch;
  padding-bottom: 16px;
}

.suggested-edits-page__track {
  display: flex;
  gap: 16px;
  align-items: stretch;
  width: max-content;
  padding-left: 0;
  padding-right: 0;
}

.suggested-edit-card {
  display: flex;
  flex-direction: column;
  width: calc(100vw - 88px);
  max-width: 30rem;
  flex: 0 0 calc(100vw - 88px);
  height: calc(100vh - 48px - 20px - 16px - 48px);
  min-height: calc(100vh - 48px - 20px - 16px - 48px);
  border: 1px solid var(--border-color-subtle);
  border-radius: var(--border-radius-base);
  background-color: var(--background-color-base);
  overflow: hidden;
  opacity: 0.72;
  scroll-snap-align: start;
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
  flex: 1 1 auto;
  min-height: 0;
  overflow: auto;
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
  background-color: var(--background-color-neutral);
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
  white-space: normal;
}

.suggested-edit-card__link,
.suggested-edit-card__excerpt :deep(a) {
  color: var(--color-progressive);
  text-decoration: none;
}

.suggested-edit-card__excerpt :deep(.suggested-edit-card__highlight) {
  display: inline-block;
  height: 24px;
  padding: 0 4px;
  background-color: #d9e2ff;
  line-height: 24px;
  box-decoration-break: clone;
  -webkit-box-decoration-break: clone;
  vertical-align: baseline;
}

.suggested-edit-card__panel {
  flex-shrink: 0;
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
  padding: 0 8px;
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

.suggested-edit-card__success-icon :deep(.cdx-icon) {
  color: var(--color-inverted);
}

.progression-item__marker :deep(.cdx-icon) {
  width: 12px;
  height: 12px;
  color: var(--color-inverted);
}

.progression-item__title {
  color: var(--color-base);
  font-size: 16px;
  line-height: 22px;
}

.progression-item__description,
.progression-item__duration {
  color: var(--color-subtle);
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
  background-color: var(--background-color-base) !important;
}

.module-card-link:deep(.cdx-card__wrapper),
.module-card-link:deep(.cdx-card__text) {
  background-color: var(--background-color-base) !important;
}

.module-card-link:deep(.cdx-card__text__title),
.module-card-link:deep(.cdx-card__text__description) {
  background-color: transparent;
}

.module-card-link:deep(.cdx-card__text__description) {
  color: var(--color-subtle);
  font-size: 14px;
  line-height: 20px;
}

.progression-item__status :deep(.cdx-icon) {
  color: var(--color-success);
}

.progression-item--recently-completed .progression-item__marker {
  animation: progression-marker-complete 420ms ease;
}

@media (max-width: 639px) {
  .contributor-sheet__panel {
    width: 100%;
    max-width: none;
    border-radius: 12px 12px 0 0;
  }
}

.contributor-sheet-fade-enter-active,
.contributor-sheet-fade-leave-active {
  transition: opacity 180ms ease;
}

.contributor-sheet-fade-enter-active .contributor-sheet__panel,
.contributor-sheet-fade-leave-active .contributor-sheet__panel {
  transition: transform 220ms ease;
}

.contributor-sheet-fade-enter-from,
.contributor-sheet-fade-leave-to {
  opacity: 0;
}

.contributor-sheet-fade-enter-from .contributor-sheet__panel,
.contributor-sheet-fade-leave-to .contributor-sheet__panel {
  transform: translateY(24px);
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

@keyframes homepage-level-chip-glow {
  0% {
    transform: scale(1);
  }

  35% {
    transform: scale(1.04);
  }

  60% {
    transform: scale(1.02);
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
  background-color: var(--background-color-progressive-subtle);
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

  .survey-page__footer-nav-inner,
  .quiz-page__footer-nav-inner {
    max-width: 33rem;
    padding: 0 var(--spacing-100);
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

  .suggested-edit-card {
    width: min(30rem, calc(100vw - 120px));
    flex-basis: min(30rem, calc(100vw - 120px));
  }
}
</style>
