<template>

  <CoreBase
    :immersivePage="false"
    :authorized="userIsAuthorized"
    authorizedRole="adminOrCoach"
    :showSubNav="true"
  >

    <TopNavbar slot="sub-nav" />

    <KPageContainer>
      <p>
        <BackLink
          :to="classRoute('ReportsGroupReportPage')"
          :text="group.name"
        />
      </p>
      <h1>
        <KLabeledIcon icon="lesson" :label="lesson.title" />
      </h1>
      <p>{{ $tr('lessonProgressLabel', {lesson: lesson.title}) }}</p>
      <HeaderTable>
        <HeaderTableRow :keyText="coachString('statusLabel')">
          <LessonActive slot="value" :active="lesson.active" />
        </HeaderTableRow>
        <HeaderTableRow
          :keyText="coachString('descriptionLabel')"
          :valueText="lesson.description || coachString('descriptionMissingLabel')"
        />
      </HeaderTable>

      <CoreTable :emptyMessage="coachString('lessonListEmptyState')">
        <thead slot="thead">
          <tr>
            <th>{{ coachString('titleLabel') }}</th>
            <th>{{ coreString('progressLabel') }}</th>
            <th>{{ coachString('avgTimeSpentLabel') }}</th>
          </tr>
        </thead>
        <transition-group slot="tbody" tag="tbody" name="list">
          <tr v-for="tableRow in table" :key="tableRow.node_id">
            <td>
              <KLabeledIcon :icon="tableRow.kind">
                <KRouterLink
                  v-if="tableRow.kind === 'exercise'"
                  :text="tableRow.title"
                  :to="classRoute(
                    'ReportsGroupReportLessonExerciseLearnerListPage',
                    { exerciseId: tableRow.content_id }
                  )"
                />
                <KRouterLink
                  v-else
                  :text="tableRow.title"
                  :to="classRoute(
                    'ReportsGroupReportLessonResourceLearnerListPage',
                    { resourceId: tableRow.content_id }
                  )"
                />
              </KLabeledIcon>
            </td>
            <td>
              <StatusSummary
                :tally="tableRow.tally"
                :verbose="true"
              />
            </td>
            <td>
              <TimeDuration :seconds="tableRow.avgTimeSpent" />
            </td>
          </tr>
        </transition-group>
      </CoreTable>
    </KPageContainer>
  </CoreBase>

</template>


<script>

  import commonCoreStrings from 'kolibri.coreVue.mixins.commonCoreStrings';
  import commonCoach from '../common';

  export default {
    name: 'ReportsGroupReportLessonPage',
    components: {},
    mixins: [commonCoach, commonCoreStrings],
    computed: {
      lesson() {
        return this.lessonMap[this.$route.params.lessonId];
      },
      group() {
        return this.groupMap[this.$route.params.groupId];
      },
      recipients() {
        return this.getLearnersForGroups([this.$route.params.groupId]);
      },
      table() {
        const contentArray = this.lesson.node_ids.map(node_id => this.contentNodeMap[node_id]);
        const sorted = this._.sortBy(contentArray, ['title']);
        return sorted.map(content => {
          const tableRow = {
            avgTimeSpent: this.getContentAvgTimeSpent(content.content_id, this.recipients),
            tally: this.getContentStatusTally(content.content_id, this.recipients),
          };
          Object.assign(tableRow, content);
          return tableRow;
        });
      },
    },
    $trs: {
      lessonProgressLabel: "'{lesson}' progress",
    },
  };

</script>


<style lang="scss" scoped></style>
