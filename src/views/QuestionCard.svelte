<script lang="ts">
  import Button from '../components/Button/Button.svelte';
  import IconButton from '../components/Button/IconButton.svelte';
  import Pin from '../components/icons/Pin.svelte';
  import QuestionMarkCircle from '../components/icons/QuestionMarkCircle.svelte';
  import List from '../components/List/List.svelte';
  import ListItem from '../components/List/ListItem.svelte';
  import type { ListItemVariant } from '../components/List/types';
  import Paper from '../components/Paper.svelte';
  import Rating from '../components/Rating.svelte';
  import { statsService } from '../stores/stats.svelte';
  import type { AnswerWithCorrectness, Question } from '../types';
  import type { LicenseId } from '../utils/licenses';
  import { shuffleAnswers } from '../utils/shuffleAnswers';

  let {
    licenseId,
    question,
    hasPreviousQuestion,
    hasNextQuestion,
    onNextQuestionClick,
    onPreviousQuestionClick,
    onPinClick,
  }: {
    licenseId: LicenseId;
    question: Question;
    hasPreviousQuestion: boolean;
    hasNextQuestion: boolean;
    onNextQuestionClick: () => void;
    onPreviousQuestionClick: () => void;
    onPinClick: () => void;
  } = $props();

  let selectedAnswerIndex = $state<number | undefined>(undefined);
  let shuffledAnswers = $derived(shuffleAnswers(question.answers));
  let stats = $derived(statsService.getQuestionStats(licenseId, question.id));

  function answerQuestion(index: number) {
    selectedAnswerIndex = index;

    if (shuffledAnswers[index].isCorrect) {
      statsService.registerCorrectAnswer(licenseId, question.id);
    } else {
      statsService.registerWrongAnswer(licenseId, question.id);
    }
  }

  function getAnswerColorVariant(answer: AnswerWithCorrectness, index: number): ListItemVariant {
    if (selectedAnswerIndex !== undefined) {
      if (answer.isCorrect) {
        return 'success';
      } else if (index === selectedAnswerIndex) {
        return 'error';
      }
    }

    return 'info';
  }

  function reveal() {
    selectedAnswerIndex = -1;
  }

  function onSelection(index: number) {
    if (selectedAnswerIndex === undefined) {
      answerQuestion(index);
    } else {
      onNextQuestionClick();
    }
  }

  function selectAnswer(event: KeyboardEvent) {
    if (event.key === '1') {
      onSelection(0);
    } else if (event.key === '2') {
      onSelection(1);
    } else if (event.key === '3') {
      onSelection(3);
    } else if (event.key === '4') {
      onSelection(3);
    } else if (event.key === 'Enter') {
      onSelection(-1);
    }
  }
</script>

<svelte:window onkeydown={selectAnswer} />

<Paper class="divide-y divide-slate-200 bg-white">
  <div class="relative min-h-[100px] rounded-t-lg bg-blue-100 p-5">
    <div class="flex items-start gap-3">
      <div class="pt-1 text-blue-600"><QuestionMarkCircle size="lg" /></div>
      <h2 class="text-xl font-medium text-pretty text-blue-900">{question?.text}</h2>
    </div>

    {#if question.images.length}
      <div class="align-center mt-5 flex flex-row justify-center gap-2">
        {#each question.images as fileName}
          <img
            class="pointer-events-none select-none"
            src={`../../generated/assets/${fileName}`}
            alt=""
          />
        {/each}
      </div>
    {/if}

    <IconButton
      label={stats?.pinned ? 'Unpin' : 'Pin'}
      class="absolute right-0 bottom-0 translate-x-[50%] translate-y-[50%] shadow-lg"
      variant={stats?.pinned ? 'filled' : 'white'}
      onclick={onPinClick}><Pin size="lg" /></IconButton
    >
  </div>

  <List class="p-6">
    {#each shuffledAnswers as answer, index (answer.text)}
      <ListItem
        variant={getAnswerColorVariant(answer, index)}
        onclick={() => onSelection(index)}>{answer.text}</ListItem
      >
    {/each}
  </List>

  <div class="p-4">
    <div class="flex w-full items-center justify-between gap-4">
      <div class="flex-1 self-start">
        {#if hasPreviousQuestion}
          <Button
            variant="text"
            onclick={onPreviousQuestionClick}>Vorherige</Button
          >
        {/if}
      </div>

      {#key question.id}
        <Rating progress={stats?.progress} />
      {/key}

      <div class="flex flex-1 justify-end">
        {#if selectedAnswerIndex === undefined}
          <Button
            variant="text"
            onclick={reveal}>Aufdecken</Button
          >
        {:else if hasNextQuestion}
          <Button onclick={onNextQuestionClick}>Weiter</Button>
        {:else}
          <Button onclick={onNextQuestionClick}>Beenden</Button>
        {/if}
      </div>
    </div>
  </div>
</Paper>
