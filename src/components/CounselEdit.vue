<template>
  <v-app>
    <!-- <CommonToolbar /> -->
    <v-content>
      <v-layout>
        <!-- <CommonLmenu /> -->
        <v-card min-height="860" min-width="1024" width="100%">
          <v-breadcrumbs :items="navigations" class="pa-2">
            <template v-slot:divider>
              <v-icon>mdi-chevron-right</v-icon>
            </template>
          </v-breadcrumbs>

          <v-container fluid class="py-0">
            <v-row justify="center" class="my-2">
              <v-col cols="6" md="4">
                <!-- 프론트 접근 설정 -->
                <v-dialog v-if="mode === 'view'" v-model="showFrontDialog" width="500">
                  <template v-slot:activator="{ on }">
                    <v-btn color="red lighten-2" dark v-on="on">
                      화면 접근 설정&nbsp;
                      <b>[{{isBlockFront ? 'ON' : 'OFF'}}]</b>
                    </v-btn>
                  </template>

                  <v-card flat>
                    <v-card-title class="headline grey lighten-2" primary-title>화면 접근 설정</v-card-title>
                    <v-card-text>
                      <v-container fluid>
                        <v-row justify="center">
                          <v-switch inset color="error" v-model="frontSwitch" :label="'ON'">
                            <template #prepend>
                              <v-label>OFF</v-label>
                            </template>
                          </v-switch>
                        </v-row>
                        <p class="text-center">
                          <v-icon style="color:red;">mdi-alert-outline</v-icon>
                          <b>[ON]</b> 으로 변경 시 홈페이지 상담 예약 신청 화면에
                          <br />사용자는 접근할 수 없습니다.
                        </p>
                      </v-container>
                    </v-card-text>
                    <v-divider></v-divider>

                    <v-card-actions>
                      <v-btn text @click="showFrontDialog = false">취소</v-btn>
                      <v-spacer></v-spacer>
                      <v-btn color="primary" text @click="changeFrontSwitch">확인</v-btn>
                    </v-card-actions>
                  </v-card>
                </v-dialog>
              </v-col>
              <v-col cols="12" sm="6" md="6">
                <v-icon
                  class="ma-1 mdi-36px"
                  v-if="mode === 'view'"
                  @click="prevWeek()"
                >mdi-chevron-left</v-icon>
                <span class="ma-2 title">
                  {{$moment(dateStart).format('YYYY-MM-DD')}}
                  <v-icon class="ma-1">mdi-minus</v-icon>
                  {{$moment(dateEnd).format('YYYY-MM-DD')}}
                </span>
                <v-icon
                  class="ma-1 mdi-36px"
                  v-if="mode === 'view'"
                  @click="nextWeek()"
                >mdi-chevron-right</v-icon>
              </v-col>
              <v-col cols="6" md="2">
                <!-- 수정 버튼 -->
                <v-btn
                  class="ma-1"
                  color="secondary"
                  v-if="mode === 'view'"
                  @click="changeEditMode('edit')"
                >
                  <v-icon class="mdi-18px">mdi-table-edit</v-icon>&nbsp;수정
                </v-btn>
              </v-col>
            </v-row>

            <v-simple-table class="table-border" v-if="!isLoading">
              <template v-slot:default>
                <tbody>
                  <tr class="table-header">
                    <td rowspan="2" width="200">구분</td>
                    <template v-if="days && days.length > 0">
                      <td
                        v-for="(day, index) in days"
                        :key="'day'+index"
                        :class="day.type === 'H' ? 'red--text' : ''"
                      >{{day.dayNm}}</td>
                    </template>
                    <template v-else>
                      <td class="red--text">월</td>
                      <td>화</td>
                      <td>수</td>
                      <td>목</td>
                      <td>금</td>
                      <td>토</td>
                      <td>일</td>
                    </template>
                  </tr>

                  <tr class="table-header">
                    <template v-if="days && days.length > 0">
                      <td
                        v-for="(day, index) in days"
                        :key="'day'+index"
                        :class="day.type === 'H' ? 'red--text' : ''"
                      >{{$moment(day.ymd).format('YYYY-MM-DD')}}</td>
                    </template>
                    <template v-else>
                      <td class="red--text">{{$moment(dateStart).format('YYYY-MM-DD')}}</td>
                      <td>{{$moment(dateStart).add(1, 'day').format('YYYY-MM-DD')}}</td>
                      <td>{{$moment(dateStart).add(2, 'day').format('YYYY-MM-DD')}}</td>
                      <td>{{$moment(dateStart).add(3, 'day').format('YYYY-MM-DD')}}</td>
                      <td>{{$moment(dateStart).add(4, 'day').format('YYYY-MM-DD')}}</td>
                      <td>{{$moment(dateStart).add(5, 'day').format('YYYY-MM-DD')}}</td>
                      <td>{{$moment(dateEnd).format('YYYY-MM-DD')}}</td>
                    </template>
                  </tr>

                  <tr v-for="(time, index) in times" :key="'time'+index">
                    <td class="text-center">{{time.text}}</td>
                    <td v-for="(counsel, key, idx) in dayCounselInfo" :key="'data'+counsel.ymd+idx">
                      <v-form v-model="formValid" v-if="mode === 'edit'">
                        <v-text-field
                          type="number"
                          min="0"
                          :rules="[countRules.valid, countRules.count(counsel[time.type]['rCnt'])]"
                          v-model="counsel[time.type]['count']"
                          hide-details
                          dense
                          outlined
                        ></v-text-field>
                      </v-form>
                      <p v-else class="text-center">{{ counsel[time.type]['count'] }}</p>
                    </td>
                  </tr>

                  <tr>
                    <td class="text-center">1일 상담 건수</td>
                    <td v-for="(dayCount, idx) in dayCounselCount" :key="'dayCount'+idx">
                      <p class="text-center" v-text="dayCount"></p>
                    </td>
                  </tr>

                  <tr>
                    <td class="text-center">주 상담 건수</td>
                    <td colspan="7" align="center">
                      <p class="text-center" v-text="weekCounselCount"></p>
                    </td>
                  </tr>
                </tbody>
              </template>
            </v-simple-table>

            <v-row justify="center" class="my-2" v-if="mode === 'edit'">
              <v-btn rounded tile color="secondary" @click="changeEditMode('view')">
                <v-icon class="mdi-18px">mdi-delete-forever</v-icon>&nbsp;취소
              </v-btn>&nbsp;
              <v-btn rounded tile color="secondary" @click="save()">
                <v-icon class="mdi-18px">mdi-content-save</v-icon>&nbsp;저장
              </v-btn>
            </v-row>
          </v-container>
        </v-card>
      </v-layout>

      <v-snackbar v-model="snackbar" :timeout="2500" :color="snackbar_color">
        {{ snackbar_text }}
        <v-btn dark text :top="true" @click="snackbar = false">닫기</v-btn>
      </v-snackbar>
    </v-content>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      mode: "view",
      formValid: false,
      isLoading: true,
      showFrontDialog: false, // 프론트 접근 설정 다이얼로그
      isBlockFront: false, // 프론트 접근 제어 여부
      frontSwitch: false, // 다이얼로그에서 스위치 선택 현황
      snackbar: false,
      snackbar_color: "",
      snackbar_text: "",
      todayYear: this.$moment()
        .day(1)
        .format("YYYY"),
      todayMonth: this.$moment()
        .day(1)
        .format("MM"),
      currentIdx: 0, // 주간 날짜 계산용
      dateStart: "",
      dateEnd: "",
      days: [], // 주간 날짜
      times: [
        { text: "10:30 ~ 12:00", type: 1 },
        { text: "13:00 ~ 14:30", type: 2 },
        { text: "15:00 ~ 16:30", type: 3 },
        { text: "17:00 ~ 18:30", type: 4 }
      ],
      countRules: {
        valid: v => {
          return (typeof Number.parseInt(v) === "number" && v > -1) || false;
        },
        count(rCnt) {
          return v => v >= rCnt || "초과";
        }
      }, // 횟수 유효성 체크
      holidays: [], // 휴일 데이터
      dayCounselInfo: {}, // 날짜별 상담 데이터
      tmpCounselInfo: {}, // 수정 페이지 리셋용
      counselList: [] // DB 저장된 데이터
    };
  },

  computed: {
    navigations() {
      return [
        {
          text: "홈",
          href: "/#/views/index"
        },
        {
          text: "일정관리",
          disabled: true
        },
        {
          text: "상담 가능일자 설정",
          disabled: true
        }
      ];
    },
    /**
     * 1일 상담 건수 배열
     */
    dayCounselCount() {
      let arr = [];
      for (let [key, value] of Object.entries(this.dayCounselInfo)) {
        let cnt = 0;
        for (let [k, v] of Object.entries(value)) {
          cnt += Number.parseInt(v.count);
        }
        arr.push(cnt);
      }
      return arr;
    },
    /**
     * 주 상담 건수
     */
    weekCounselCount() {
      let cur = 0;
      return this.dayCounselCount.reduce((acc, cur) => acc + cur);
    },
    /**
     * 현재 상담 데이터 존재 여부
     */
    isCounselData() {
      return this.counselList && this.counselList.length > 0;
    }
  },

  created() {
    this.calDate();
    this.getFrontBlockStatus();
    this.holidays = this.$utils.getHolidays();
  },

  methods: {
    accessMode() {
      return ["admin"];
    },

    /**
     * 프론트 화면 접근 상태 가져오기
     */
    getFrontBlockStatus() {
      this.$sendApi("/code/counselOnOff")
        .then(response => {
          if (response.data && response.data.isSuccess) {
            const flag = response.data.code.value === "ON" ? true : false;
            // 접근 설정 flag
            this.isBlockFront = flag;
            // 다이얼로그용
            this.frontSwitch = flag;
          }
        })
        .catch(error => {
          console.log("ERROR::", error);
          this.openSnackbar(0, "화면 접근 상태 조회 중 오류가 발생했습니다.");
        });
    },

    /**
     * 주간 날짜 계산
     */
    calDate() {
      this.days = [];
      const dateToday = this.$moment().add(this.currentIdx, "days");
      // 주간 시작일
      this.dateStart = dateToday.day(1).format("YYYYMMDD");
      this.todayYear = dateToday.day(1).format("YYYY");
      this.todayMonth = dateToday.day(1).format("MM");
      // 주간 종료일
      this.dateEnd = dateToday
        .day(1)
        .add(6, "days")
        .format("YYYYMMDD");

      let days = [];
      for (let i = 0; i < 7; i++) {
        const momentDay = this.$moment(this.dateStart);
        days.push({
          ymd: momentDay.add(i, "day").format("YYYYMMDD"),
          dayNm: momentDay.locale("ko").format("ddd") // 요일
        });
      }
      this.days = days;
      this.getCounselData();
    },

    /**
     * 상담 데이터 가져오기
     */
    getCounselData() {
      const data = {
        bgnDt: this.dateStart,
        endDt: this.dateEnd
      };
      this.$sendApi("/counsel/counselList", data)
        .then(response => {
          if (response.data.isSuccess == true) {
            // console.log(response.data)
            this.counselList = response.data.counselList;
            this.fetchData();
          }
        })
        .catch(error => {
          console.log("ERROR::", error);
          this.openSnackbar(0, "데이터 조회 중 오류가 발생했습니다.");
        });
    },

    /**
     * 데이터 처리
     */
    fetchData() {
      this.days = this.days.map(day => {
        let type = "D"; // 주간
        if (day.dayNm === "토" || day.dayNm === "일") type = "W"; // 주말
        if (day.dayNm === "월") type = "H"; // 월요일 휴관
        if (this.holidays.includes(day.ymd)) type = "H"; // 공휴일 체크
        day.type = type;
        return day;
      });
      this.createCounselInfo();
    },

    /**
     * 카운셀 데이터 초기값 설정
     */
    createCounselInfo() {
      let dataMap = {};
      this.days.forEach(day => {
        let dayData = {};
        this.times.forEach((time, index) => {
          let count = 0;
          let rCnt = 0;
          // 수정화면에서 데이터 없을 경우에만 초기 데이터 셋팅
          if (this.mode === "edit" && !this.isCounselData) {
            if (day.type === "D" && index > 1) count = 2;
            if (day.type === "W" && index > 0) count = 3;
          } else if (this.isCounselData) {
            const data = this.counselList.filter(d => {
              return d.counselDay === day.ymd && d.timePeriod === index + 1;
            })[0];
            if (data) {
              count =
                Number.parseInt(data.aCnt) + Number.parseInt(data.rCnt) || 0;
              rCnt = Number.parseInt(data.rCnt) || 0;
            }
          }

          dayData[time.type] = {
            count: count,
            rCnt: rCnt // 현재 예약되어있는 개수
          };
        });
        dataMap[day.ymd] = dayData;
      });
      this.dayCounselInfo = dataMap;
      this.isLoading = false;
    },

    /**
     * 상담 가능 일자 설정 저장
     */
    save() {
      // validation
      if (!this.formValid) {
        this.openSnackbar(0, "데이터를 확인해주세요.");
        return;
      }

      if (this.mode !== "edit" || !confirm("저장하시겠습니까?")) return;
      let counselData = [];
      for (let [key, value] of Object.entries(this.dayCounselInfo)) {
        for (let [k, v] of Object.entries(value)) {
          // 유효성 체크
          if (v < 0) continue;
          counselData.push({
            counselDay: key,
            timePeriod: k,
            counselTotal: v.count
          });
        }
      }
      // console.log(counselData);

      this.$sendApi("/counsel/counselInsertProc", { counselData: counselData })
        .then(response => {
          if (response.data.isSuccess == true) {
            // console.log(response.data);
            this.mode = "view";
            this.openSnackbar(1, "Success");
            this.getCounselData();
          }
        })
        .catch(error => {
          console.log("ERROR::", error);
          this.openSnackbar(0, "데이터 저장 중 오류가 발생했습니다.");
        });
    },

    /**
     * 프론트 화면 접근 설정 변경
     */
    changeFrontSwitch() {
      const params = {
        field: "ON_OFF",
        type: "COUNSEL",
        value: this.frontSwitch ? "ON" : "OFF"
      };
      this.$sendApi("/code/counselOnOffProc", params)
        .then(response => {
          if (response.data.isSuccess == true) {
            // console.log(response.data);
            this.mode = "view";
            this.openSnackbar(1, "Success");
            this.getFrontBlockStatus();
            this.showFrontDialog = false;
          }
        })
        .catch(error => {
          console.log("ERROR::", error);
          this.openSnackbar(0, "데이터 저장 중 오류가 발생했습니다.");
        });
    },

    prevWeek() {
      // 2020년 6월 서비스 개시로 그 전으로는 이동 불가
      if (
        Number.parseInt(this.todayYear) <= 2020 &&
        Number.parseInt(this.todayMonth) <= 5
      ) {
        return;
      }
      this.currentIdx -= 7;
      this.calDate();
    },

    nextWeek() {
      this.currentIdx += 7;
      this.calDate();
    },

    resetData() {
      this.dayCounselInfo = _.cloneDeep(this.tmpCounselInfo);
    },

    changeEditMode(mode) {
      this.mode = mode;
      if (mode === "view") {
        this.resetData();
      } else {
        // 초기화할 데이터 저장
        this.tmpCounselInfo = _.cloneDeep(this.dayCounselInfo);
        this.createCounselInfo();
      }
    },

    /* 알림 메시지 */
    openSnackbar(result, msg) {
      if (result == 1) {
        this.snackbar = true;
        this.snackbar_text = msg;
        this.snackbar_color = "success";
      } else {
        this.snackbar = true;
        this.snackbar_text = msg;
        this.snackbar_color = "error";
      }
    }
  }
};
</script>

<style scoped>
.table-header {
  background-color: #eeeeee;
  width: 200px;
  font-weight: bold;
  text-align: center;
}
.table-border {
  border: 1px solid #eeeeee !important;
}
</style>
