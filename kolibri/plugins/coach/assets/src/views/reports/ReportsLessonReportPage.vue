<template>

  <CoreBase
    :immersivePage="false"
    :authorized="userIsAuthorized"
    authorizedRole="adminOrCoach"
    :showSubNav="true"
  >

    <TopNavbar slot="sub-nav" />
    <KGrid gutter="16">
      <KGridItem>
        <QuizLessonDetailsHeader
          examOrLesson="lesson"
          :backlinkLabel="coreString('allLessonsLabel')"
          :backlink="classRoute('ReportsLessonListPage')"
        >
          <LessonOptionsDropdownMenu
            slot="dropdown"
            optionsFor="report"
            @select="handleSelectOption"
          />
        </QuizLessonDetailsHeader>
      </KGridItem>
      <KGridItem :layout12="{ span: 4 }">
        <LessonStatus
          activeKey="active"
          :lesson="lesson"
          :groupNames="getGroupNames(lesson.groups)"
        />
      </KGridItem>
      <KGridItem :layout12="{ span: 8 }">
        <KPageContainer>


          <HeaderTabs>

            <HeaderTab
              :text="coachString('reportLabel')"
              :to="classRoute('ReportsLessonReportPage', {})"
            />
            <HeaderTab
              :text="coreString('learnersLabel')"
              :to="classRoute('ReportsLessonLearnerListPage', {})"
            />
          </HeaderTabs>
          <CoreTable :emptyMessage="emptyMessage">
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
                      v-if="tableRow.kind === 'exercise' && tableRow.hasAssignments"
                      :text="tableRow.title"
                      :to="classRoute(
                        'ReportsLessonExerciseLearnerListPage',
                        { exerciseId: tableRow.content_id }
                      )"
                    />
                    <KRouterLink
                      v-else-if="tableRow.hasAssignments"
                      :text="tableRow.title"
                      :to="classRoute(
                        'ReportsLessonResourceLearnerListPage',
                        { resourceId: tableRow.content_id }
                      )"
                    />
                    <template v-else>
                      {{ tableRow.title }}
                    </template>
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
      </KGridItem>
    </KGrid>


  </CoreBase>

</template>


<script>

  import commonCoreStrings from 'kolibri.coreVue.mixins.commonCoreStrings';
  import commonCoach from '../common';
  import LessonOptionsDropdownMenu from '../plan/LessonSummaryPage/LessonOptionsDropdownMenu';

  export default {
    name: 'ReportsLessonReportPage',
    components: {
      LessonOptionsDropdownMenu,
    },
    mixins: [commonCoach, commonCoreStrings],
    computed: {
      emptyMessage() {
        return this.coachString('noResourcesInLessonLabel');
      },
      lesson() {
        return this.lessonMap[this.$route.params.lessonId];
      },
      recipients() {
        return this.getLearnersForLesson(this.lesson);
      },
      table() {
        const contentArray = this.lesson.node_ids.map(node_id => this.contentNodeMap[node_id]);
        const sorted = this._.sortBy(contentArray, ['title']);
        return sorted.map(content => {
          const tally = this.getContentStatusTally(content.content_id, this.recipients);
          const tableRow = {
            avgTimeSpent: this.getContentAvgTimeSpent(content.content_id, this.recipients),
            tally,
            hasAssignments: Object.values(tally).reduce((a, b) => a + b, 0),
          };
          Object.assign(tableRow, content);
          return tableRow;
        });
      },
    },
    methods: {
      handleSelectOption(action) {
        if (action === 'EDIT_DETAILS') {
          this.$router.push(this.$router.getRoute('LessonReportEditDetailsPage'));
        }
        if (action === 'MANAGE_RESOURCES') {
          this.$router.push(
            this.$router.getRoute(
              'SELECTION_ROOT',
              {},
              // So the "X" and "Cancel" buttons return back to the ReportPage
              { last: this.$route.name }
            )
          );
        }
      },
    },
    $trs: {},
  };

</script>


<style lang="scss" scoped></style>
