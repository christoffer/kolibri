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

          <h2>{{ coachString('overallLabel') }}</h2>

          <CoreTable :emptyMessage="coachString('learnerListEmptyState')">
            <thead slot="thead">
              <tr>
                <th>{{ coachString('nameLabel') }}</th>
                <th>{{ coreString('progressLabel') }}</th>
                <th>{{ coachString('groupsLabel') }}</th>
              </tr>
            </thead>
            <transition-group slot="tbody" tag="tbody" name="list">
              <tr v-for="tableRow in table" :key="tableRow.id">
                <td>
                  <KLabeledIcon icon="person">
                    <KRouterLink
                      :text="tableRow.name"
                      :to="classRoute('ReportsLessonLearnerPage', { learnerId: tableRow.id })"
                    />
                  </KLabeledIcon>
                </td>
                <td>
                  <StatusSimple :status="tableRow.status" />
                </td>
                <td>
                  <TruncatedItemList :items="tableRow.groups" />
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
    name: 'ReportsLessonLearnerListPage',
    components: {
      LessonOptionsDropdownMenu,
    },
    mixins: [commonCoach, commonCoreStrings],
    computed: {
      lesson() {
        return this.lessonMap[this.$route.params.lessonId];
      },
      recipients() {
        return this.getLearnersForLesson(this.lesson);
      },
      table() {
        const learners = this.recipients.map(learnerId => this.learnerMap[learnerId]);
        const sorted = this._.sortBy(learners, ['name']);
        return sorted.map(learner => {
          const tableRow = {
            groups: this.getGroupNamesForLearner(learner.id),
            status: this.getLessonStatusStringForLearner(this.lesson.id, learner.id),
          };
          Object.assign(tableRow, learner);
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
