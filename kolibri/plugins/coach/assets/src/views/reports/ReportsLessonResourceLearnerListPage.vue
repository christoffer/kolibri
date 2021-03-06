<template>

  <CoreBase
    :immersivePage="false"
    :authorized="userIsAuthorized"
    authorizedRole="adminOrCoach"
    :showSubNav="true"
  >

    <TopNavbar slot="sub-nav" />

    <KPageContainer>
      <section>
        <HeaderWithOptions>
          <BackLink
            slot="header"
            :to="classRoute('ReportsLessonReportPage', {})"
            :text="coachString('backToLessonLabel', { lesson: lesson.title })"
          />
          <KButton
            slot="options"
            :text="coachString('previewAction')"
            @click="onPreviewClick"
          />
        </HeaderWithOptions>
        <h1>
          <KLabeledIcon :icon="resource.kind" :label="resource.title" />
        </h1>
      </section>

      <KCheckbox
        :label="coachString('viewByGroupsLabel')"
        :checked="viewByGroups"
        @change="toggleGroupsView"
      />

      <div v-if="viewByGroups">
        <div
          v-for="group in lessonGroups"
          :key="group.id"
          :data-test="`group-${group.id}`"
          class="group"
        >
          <h2
            class="group-title"
            data-test="group-title"
          >
            <KLabeledIcon icon="group" :label="group.name" />
          </h2>

          <KFixedGrid numCols="2">
            <KFixedGridItem span="1">
              <StatusSummary
                :tally="getGroupTally(group.id)"
                :showNeedsHelp="false"
                :verbose="false"
              />
            </KFixedGridItem>
            <KFixedGridItem span="1">
              <ReportsResourcesStats :avgTime="getGroupRecipientsAvgTime(group.id)" />
            </KFixedGridItem>
          </KFixedGrid>

          <ReportsResourceLearners
            :entries="getGroupEntries(group.id)"
            :showGroupsColumn="false"
          />
        </div>

        <div
          v-if="ungroupedEntries.length"
          class="group"
        >
          <h2
            class="group-title"
            data-test="group-title"
          >
            {{ coachString('ungroupedLearnersLabel') }}
          </h2>

          <ReportsResourceLearners
            :entries="ungroupedEntries"
            :showGroupsColumn="false"
          />
        </div>
      </div>

      <template v-else>
        <ReportsResourcesStats
          :avgTime="allRecipientsAvgTime"
          data-test="summary-resources-stats"
        />

        <p>
          <StatusSummary
            :tally="summaryTally"
            data-test="summary-tally"
          />
        </p>

        <ReportsResourceLearners :entries="allEntries" />
      </template>
    </KPageContainer>
  </CoreBase>

</template>


<script>

  import { LastPages } from '../../constants/lastPagesConstants';
  import commonCoach from '../common';
  import HeaderWithOptions from '../common/HeaderWithOptions';
  import ReportsResourceLearners from './ReportsResourceLearners';
  import ReportsResourcesStats from './ReportsResourcesStats';

  export default {
    name: 'ReportsLessonResourceLearnerListPage',
    components: {
      HeaderWithOptions,
      ReportsResourceLearners,
      ReportsResourcesStats,
    },
    mixins: [commonCoach],
    data() {
      return {
        viewByGroups: Boolean(this.$route.query.groups),
      };
    },
    computed: {
      lesson() {
        return this.lessonMap[this.$route.params.lessonId];
      },
      resource() {
        return this.contentMap[this.$route.params.resourceId];
      },
      recipients() {
        return this.getLearnersForLesson(this.lesson);
      },
      allRecipientsAvgTime() {
        return this.getContentAvgTimeSpent(this.$route.params.resourceId, this.recipients);
      },
      summaryTally() {
        return this.getContentStatusTally(this.$route.params.resourceId, this.recipients);
      },
      lessonGroups() {
        if (!this.lesson.groups.length) {
          return this.groups;
        }

        return this.groups.filter(group => this.lesson.groups.includes(group.id));
      },
      allEntries() {
        const learners = this.recipients.map(learnerId => this.learnerMap[learnerId]);
        const sorted = this._.sortBy(learners, ['name']);
        return sorted.map(learner => {
          const tableRow = {
            groups: this.getLearnerLessonGroups(learner.id),
            statusObj: this.getContentStatusObjForLearner(
              this.$route.params.resourceId,
              learner.id
            ),
          };
          Object.assign(tableRow, learner);
          return tableRow;
        });
      },
      ungroupedEntries() {
        return this.allEntries.filter(entry => !entry.groups || !entry.groups.length);
      },
    },
    watch: {
      $route() {
        this.viewByGroups = Boolean(this.$route.query.groups);
      },
    },
    methods: {
      toggleGroupsView() {
        this.viewByGroups = !this.viewByGroups;

        let query;
        if (this.viewByGroups) {
          query = { ...this.$route.query, groups: 'true' };
        } else {
          query = { ...this.$route.query, groups: undefined };
        }

        this.$router.replace({ query });
      },
      getLearnerLessonGroups(learnerId) {
        return this.lessonGroups.filter(group => group.member_ids.includes(learnerId));
      },
      getGroupEntries(groupId) {
        return this.allEntries.filter(entry => {
          const entryGroupIds = entry.groups.map(group => group.id);
          return entryGroupIds.includes(groupId);
        });
      },
      getGroupTally(groupId) {
        const recipients = this.getLearnersForGroups([groupId]);
        return this.getContentStatusTally(this.$route.params.resourceId, recipients);
      },
      getGroupRecipientsAvgTime(groupId) {
        const recipients = this.getLearnersForGroups([groupId]);
        return this.getContentAvgTimeSpent(this.$route.params.resourceId, recipients);
      },
      onPreviewClick() {
        let lastPage = LastPages.RESOURCE_LEARNER_LIST;
        if (this.viewByGroups) {
          lastPage = LastPages.RESOURCE_LEARNER_LIST_BY_GROUPS;
        }

        this.$router.push(
          this.$router.getRoute(
            'RESOURCE_CONTENT_PREVIEW',
            {
              contentId: this.resource.node_id,
            },
            {
              last: lastPage,
              resourceId: this.resource.content_id,
            }
          )
        );
      },
    },
    $trs: {},
  };

</script>


<style lang="scss" scoped>

  .group:not(:first-child) {
    margin-top: 42px;
  }

  .group-title {
    margin-bottom: 42px;
  }

</style>
