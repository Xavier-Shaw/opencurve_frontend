<template>
  <div class="detail_course">
    <!-- 页面头部 -->

    <h3> {{course_id}}</h3>
    <div class="live-room">
      <el-row>
        <el-col :span="18">
          <div class="video-content">
            <vue3-video-play v-bind="videoOptions"></vue3-video-play>
          </div>
        </el-col>

        <el-col :span="6">
          <div>
            <el-row>
              <h4>Playlist</h4>
              <el-button style="margin-left: 105px; margin-top: 15px" type="primary"
                         @click="form_dialog_visible = true">
                添加章节
              </el-button>
            </el-row>
            <el-scrollbar style="height: 150px">
              <el-card class="card_item" v-for="index in this.session_cnt" :key="index"
                       @click="getSessionInfo(index)">
                <span class="card_content">Chapter {{ index }}</span>
              </el-card>
            </el-scrollbar>
          </div>

          <div>
            <el-descriptions title="Session Information" border :column="Number(1)">
              <el-descriptions-item label="Title">{{ session_info.title }}</el-descriptions-item>
              <br>
              <el-descriptions-item label="Session">{{ session_info.session }}</el-descriptions-item>
              <br>
              <el-descriptions-item label="Description">{{ session_info.description }}</el-descriptions-item>
              <br>
              <el-descriptions-item label="Score">{{ session_info.score }}</el-descriptions-item>
            </el-descriptions>
          </div>


          <div style="margin-top: 20px">
            <el-button style="margin: 0 25px" type="primary" @click="modifySessionInfo">修改信息</el-button>

            <el-button style="margin: 0 25px;"
                       type="primary"
                       @click="questionsVisible = true">
              查看题目
            </el-button>
          </div>
        </el-col>
      </el-row>

      <el-dialog v-model="form_dialog_visible" :title="formTitle" @close="cancelAddChapter">
        <el-form :model="form" ref="formRef">
          <el-form-item label="Title">
            <el-input v-model="form.title"/>
          </el-form-item>

          <el-form-item label="Session">
            <el-input-number disabled v-model="form.session"/>
          </el-form-item>

          <el-form-item label="Description">
            <el-input v-model="form.description"/>
          </el-form-item>

          <el-form-item label="Video">
            <el-upload
                :action="uploadActionUrl"
                :on-success="handleSuccess"
                accept="video/mp4,image/jpeg,image/gif,image/png"
                multiple
                :file-list="form.uploadVideo"
                :limit="1">
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-form-item>

          <el-form-item label="Score">
            <el-input-number v-model="form.score" :min="Number(1)" :max="Number(100)"/>
          </el-form-item>
        </el-form>

        <template #footer>
                  <span class="dialog_footer">
                    <el-button type="primary" @click="confirmAddChapter()">Submit</el-button>
                    <el-button @click="cancelAddChapter()">Cancel</el-button>
                  </span>
        </template>
      </el-dialog>

      <el-drawer v-model="questionsVisible"
                 direction="rtl"
      >
        <template #title>
          <h4>Questions</h4>
        </template>
        <template #default>
          <div>
            <div class="question-area" v-for="question in questions" :key="question">
              <h3>{{ question.title }}</h3>
              <el-radio-group v-model="question.correct_answer">
                <el-radio disabled v-for="choice in question.choices" :key="choice" :label="choice">{{ choice }}</el-radio>
              </el-radio-group>
              <br>
              <el-button type="primary" size="small" @click="modifyQuestion(question)">Modify Question</el-button>
            </div>
          </div>
        </template>
        <template #footer>
          <div style="flex: auto">
            <el-button type="primary" @click="question_dialog_visible = true">Add Question</el-button>
          </div>
        </template>
      </el-drawer>

      <el-dialog v-model="question_dialog_visible" :title="questionTitle" @close="cancelAddQuestion">
        <el-form :model="questionForm" ref="questionFormRef">
          <el-form-item label="Title">
            <el-input v-model="questionForm.title"/>
          </el-form-item>

          <el-form-item label="Question Type">
            <el-radio-group v-model="questionForm.questionType">
              <el-radio label="T/F">True/False question</el-radio>
              <el-radio label="Choice">Multiple choice question</el-radio>
            </el-radio-group>
          </el-form-item>

          <el-form-item label="Choices" v-if="questionForm.questionType === 'T/F'">
            <el-radio disabled label="disabled">True</el-radio>
            <el-radio disabled label="disabled">False</el-radio>
          </el-form-item>

          <el-form-item label="Choices" v-if="questionForm.questionType === 'Choice'">
            <el-input v-model="questionForm.choices[0]">
              <template #prepend>A</template>
            </el-input>
            <el-input v-model="questionForm.choices[1]">
              <template #prepend>B</template>
            </el-input>
            <el-input v-model="questionForm.choices[2]">
              <template #prepend>C</template>
            </el-input>
            <el-input v-model="questionForm.choices[3]">
              <template #prepend>D</template>
            </el-input>
          </el-form-item>

          <el-form-item label="Correct Answer" v-if="questionForm.questionType === 'T/F'">
            <el-radio-group v-model="questionForm.correctAnswer">
              <el-radio-button label="True">True</el-radio-button>
              <el-radio-button label="False">False</el-radio-button>
            </el-radio-group>
          </el-form-item>

          <el-form-item label="Correct Answer" v-if="questionForm.questionType === 'Choice'">
            <el-radio-group v-model="questionForm.correctAnswer">
              <el-radio-button :label="questionForm.choices[0]">A</el-radio-button>
              <el-radio-button :label="questionForm.choices[1]">B</el-radio-button>
              <el-radio-button :label="questionForm.choices[2]">C</el-radio-button>
              <el-radio-button :label="questionForm.choices[3]">D</el-radio-button>
            </el-radio-group>
          </el-form-item>

          <el-form-item label="Time Limit(minutes)">
            <el-input-number v-model="questionForm.time_limit" :min="1" :step="1" :precision="0"/>
          </el-form-item>
        </el-form>

        <template #footer>
                  <span class="dialog_footer">
                    <el-button type="primary" @click="confirmQuestion">Confirm</el-button>
                    <el-button @click="cancelAddQuestion()">Cancel</el-button>
                  </span>
        </template>
      </el-dialog>
    </div>

    <div>
      <el-row>
        <el-col :span="12">
          <el-button style="width: 100%; height: 50px"
                     @click="selection = 'Students\' Progress'">
            Students' Progress
          </el-button>
        </el-col>
        <el-col :span="12">
          <el-button style="width: 100%; height: 50px"
                     @click="selection = 'Comments'">
            Comments
          </el-button>
        </el-col>
      </el-row>
    </div>

    <div v-if="selection === 'Students\' Progress'" class="student_progress">
      <el-card class="student-score" v-for="student in students" :key="student">
        <el-row>
          <el-col :span="Number(7)">
            Student:
            <el-tag style="width: 220px; height: 40px; margin-left: 10px" effect="dark">
              {{ student.stu_id }}
            </el-tag>
          </el-col>
          <el-col :span="Number(4)">
            Total Score:
            <el-tag style="width: 80px; height: 40px; margin-left: 10px" effect="dark">
              {{ (parseFloat(student.video_score) + parseFloat(student.test_score)).toFixed(2) }} / {{ session_info.score }}
            </el-tag>
          </el-col>
          <el-col :span="Number(1)"></el-col>
          <el-col :span="Number(8)">
            <el-row style="margin-top: 7px">
              Video Progress:
              <el-progress :text-inside="true" :stroke-width="26" style="width: 220px; margin-left: 10px;"
                           :percentage="Number((Number(student.video_score) / (session_info.score - questions.length)).toFixed(2)) * 100"/>
            </el-row>
          </el-col>
          <el-col :span="Number(4)">
            Test Score:
            <el-tag style="width: 80px; height: 40px; margin-left: 10px" effect="dark">
              {{ Number(student.test_score) }} / {{ questions.length }}
            </el-tag>
          </el-col>
        </el-row>
      </el-card>
    </div>

    <div v-if="selection === 'Comments'" class="comment-area">
      <el-input
          style="margin-top: 10px; margin-bottom: 10px"
          v-model="commentContent"
          placeholder="Please input your comment">
        <template #append>
          <el-button type="primary" @click="postComment">发送</el-button>
        </template>
      </el-input>
      <el-timeline style="margin-top: 25px">
        <el-timeline-item v-for="comment in comments.filter((item) => item.father_comment_id === -1)" :key="comment"
                          :timestamp="comment.comment_date" placement="top">
          <div>
            <el-card class="father-comment">
              <h4 style="color: #409EFF">{{ comment.user_id }}</h4>
              <p>{{ comment.content }}</p>
              <el-popover placement="bottom" :width="600" trigger="click" @show="replyContent = ''">
                <template #reference>
                  <el-button style="margin-right: 16px">回复</el-button>
                </template>
                <el-input
                    v-model="replyContent"
                    placeholder="Please input your reply">
                  <template #append>
                    <el-button type="primary" @click="postReply(comment, comment)">发送</el-button>
                  </template>
                </el-input>
              </el-popover>
              <el-button v-if="show_sonComments.some((item) => item.comment_id === comment.id && item.show === true)"
                         type="primary" @click="show_sonComment(comment, false)">
                收起回复
              </el-button>
              <el-button v-if="show_sonComments.some((item) => item.comment_id === comment.id && item.show === false)"
                         type="primary" @click="show_sonComment(comment, true)">
                展开回复
              </el-button>
            </el-card>
            <div class="sonComment-area" style="margin-top: 10px"
                 v-if="show_sonComments.some((item) => item.comment_id === comment.id && item.show === true)">
              <el-timeline-item
                  v-for="sonComment in comments.filter((item) => item.father_comment_id === comment.id)"
                  :key="sonComment" :timestamp="sonComment.comment_date" placement="top">
                <el-card class="son-comment">
                  <el-row>
                    <h4 style="color: #409EFF">{{ sonComment.user_id }}</h4>
                    <p style="margin-top: 19px; margin-left: 8px; margin-right: 8px;"> reply to </p>
                    <h4 style="color: #409EFF"> {{ sonComment.reply_user_id }}</h4>
                  </el-row>

                  <p>{{ sonComment.content }}</p>
                  <el-popover placement="bottom" :width="600" trigger="click" @show="replyContent = ''">
                    <template #reference>
                      <el-button style="margin-right: 16px">回复</el-button>
                    </template>
                    <el-input
                        v-model="replyContent"
                        placeholder="Please input your reply">
                      <template #append>
                        <el-button type="primary" @click="postReply(comment, sonComment)">发送</el-button>
                      </template>
                    </el-input>
                  </el-popover>
                </el-card>
              </el-timeline-item>
            </div>
          </div>
        </el-timeline-item>
      </el-timeline>
    </div>
  </div>
</template>

<script>
import {reactive, ref} from "vue";
import axios from "axios";
import {ElMessage} from "element-plus";
import dayjs from "dayjs";
// import {useRoute} from 'vue-router'
// const route = useRoute()

export default {
  name: "TeacherLiveCourse",

  data() {
    let formRef = reactive(ref(null))
    let form = reactive({
      title: '',
      session: 0,
      description: '',
      uploadVideo: [],
      score: ''
    })
    let session_info = reactive({
      title: '',
      session: '',
      description: '',
      url: '',
      score: ''
    })
    let questionFormRef = reactive(ref(null))
    let questionForm = reactive({
      id: '',
      questionType: '',
      title: '',
      choices: ['', '', '', ''],
      correctAnswer: '',
      time_limit: 0
    })

    return {
      course_id: '',
      user_id: '',
      session_cnt: 0,
      currentSession: 1,
      session_info,
      formRef,
      form,
      questionFormRef,
      questionForm,
      formTitle: '添加课程章节',
      questionTitle: '添加问题',
      uploadActionUrl: 'http://localhost:8081/cloud_storage/file/video',
      form_dialog_visible: false,
      question_dialog_visible: false,
      videoUrl: '',
      videoOptions: reactive({
        width: "100%", //播放器高度
        height: "550px", //播放器高度
        color: "#409eff", //主题色
        title: "test video", //视频名称
        src: "https://www.w3schools.com/html/movie.mp4", //视频源
        muted: false, //静音
        webFullScreen: false,
        speedRate: ["0.75", "1.0", "1.25", "1.5", "2.0"], //播放倍速
        autoPlay: false, //自动播放
        loop: false, //循环播放
        mirror: false, //镜像画面
        lightOff: true, //关灯模式
        volume: 0.3, //默认音量大小
        control: true, //是否显示控制
        controlBtns: [
          "audioTrack",
          "quality",
          "speedRate",
          "volume",
          "setting",
          "pip",
          "pageFullScreen",
          "fullScreen",
        ], //显示所有按钮,
      }),
      questionsVisible: false,
      questions: [],
      students: [],

      // Comment
      comments: [],
      show_sonComments: [],
      commentContent: '',
      commentFatherId: -1,
      replyContent: '',

      selection: 'Comments'
    }
  },

  methods: {
    handleSuccess(resp) {
      console.log('视频存储url为: ' + resp.data.url)
      this.videoUrl = resp.data.url
    },

    confirmAddChapter() {
      axios({
        method: 'POST',
        url: 'http://localhost:8080/education/video/uploadVideoMeta',
        data: {
          course_id: this.$route.params.course_id,
          title: this.form.title,
          session: this.form.session,
          url: this.videoUrl,
          description: this.form.description,
          score: this.form.score
        },
        transformRequest: [function (data) {
          let str = '';
          for (let key in data) {
            str += encodeURIComponent(key) + '=' + encodeURIComponent(data[key]) + '&';
          }
          return str;
        }]
      })
          .then(response => {
            console.log(response.data)
            ElMessage({
              type: 'success',
              message: '添加章节成功！',
            })
            this.getSessionsCount()
            this.cancelAddChapter()
          })
    },

    cancelAddChapter() {
      this.form_dialog_visible = false
      this.form.title = ''
      this.form.session = this.session_cnt + 1
      this.form.description = ''
      this.form.uploadVideo = []
      this.$refs.formRef.resetFields()
      this.formTitle = '添加课程章节'
    },

    getSessionsCount() {
      axios({
        method: 'GET',
        url: 'http://localhost:8080/education/video/getSessionsCount?course_id=' + this.course_id,
      }).then(response => {
        this.session_cnt = response.data.data.session_count
        this.form.session = this.session_cnt + 1
      })
    },

    getSessionInfo(session) {
      axios({
        method: 'GET',
        url: 'http://localhost:8080/education/video/getSessionInfo?course_id=' + this.course_id + '&session=' + session,
      }).then(response => {
        let resp = response.data.data.video
        this.session_info = resp
        this.videoOptions.src = resp.url
        this.currentSession = resp.session
        this.getTestBySession()
        this.getStudentProgressBySession()
        this.getComment()
      })
    },

    modifySessionInfo() {
      this.formTitle = '修改课程章节'
      this.form_dialog_visible = true
      this.form.title = this.session_info.title
      this.form.session = this.session_cnt + 1
      this.form.description = this.session_info.description
      this.form.score = this.session_info.score
    },

    getTestBySession() {
      axios({
        method: 'GET',
        url: 'http://localhost:8080/education/video/getAllTestByCourseAndSession?course_id='
            + this.course_id + '&session=' + this.currentSession,
      }).then(response => {
        this.questions = response.data.data.Tests
        for (let question of this.questions) {
          question.choices = question.choices.split(',')
        }
      })
    },

    modifyQuestion(question) {
      console.log(question)
      this.question_dialog_visible = true
      this.questionTitle = '修改问题'
      this.questionForm.id = question.id
      this.questionForm.title = question.title
      this.questionForm.questionType = question.questionType
      this.questionForm.choices = question.choices
      this.questionForm.correctAnswer = question.correctAnswer
      this.questionForm.time_limit = question.time_limit
    },

    cancelAddQuestion() {
      this.question_dialog_visible = false
      this.questionForm.title = ''
      this.questionForm.questionType = ''
      this.questionForm.choices = ['', '', '', '']
      this.questionForm.correctAnswer = ''
      this.questionForm.time_limit = 0
      this.$refs.questionFormRef.resetFields()
      this.questionTitle = '添加问题'
    },

    confirmQuestion() {
      if (this.questionTitle === '添加问题') {
        this.confirmAddQuestion()
      }
      else if (this.questionTitle === '修改问题') {
        this.confirmModifyQuestion()
      }
    },

    confirmAddQuestion() {
      if (this.questionForm.questionType === 'T/F') {
        this.questionForm.choices = ['True', 'False']
      }
      axios({
        method: 'POST',
        url: 'http://localhost:8080/education/video/postTest',
        data: {
          course_id: this.course_id,
          session: this.currentSession,
          questionType: this.questionForm.questionType,
          title: this.questionForm.title,
          choices: this.questionForm.choices.toString().replace('[','').replace(']',''),
          correct_answer: this.questionForm.correctAnswer,
          time_limit: this.questionForm.time_limit
        },
        transformRequest: [function (data) {
          let str = '';
          for (let key in data) {
            str += encodeURIComponent(key) + '=' + encodeURIComponent(data[key]) + '&';
          }
          return str;
        }]
      })
          .then(response => {
            console.log(response.data)
            ElMessage({
              type: 'success',
              message: '添加问题成功！',
            })
            this.getTestBySession()
            this.cancelAddQuestion()
          })
    },

    confirmModifyQuestion() {
      axios({
        method: 'POST',
        url: 'http://localhost:8080/education/video/updateTest',
        data: {
          id: this.questionForm.id,
          questionType: this.questionForm.questionType,
          title: this.questionForm.title,
          choices: this.questionForm.choices.toString().replace('[','').replace(']',''),
          correct_answer: this.questionForm.correctAnswer,
          time_limit: this.questionForm.time_limit
        },
        transformRequest: [function (data) {
          let str = '';
          for (let key in data) {
            str += encodeURIComponent(key) + '=' + encodeURIComponent(data[key]) + '&';
          }
          return str;
        }]
      })
          .then(response => {
            console.log(response.data)
            ElMessage({
              type: 'success',
              message: '修改问题成功！',
            })
            this.getTestBySession()
            this.cancelAddQuestion()
          })
    },

    getStudentProgressBySession() {
      axios({
        method: 'GET',
        url: 'http://localhost:8080/education/video/getAllScore?course_id='
            + this.course_id + '&session=' + this.currentSession,
      }).then(response => {
        console.log(response)
        // {id: 37, stu_id: '12012902@mail.sustech.edu.cn', sess_id: 6, video_score: '0.00', text_score: null}
        this.students = response.data.data.scores
      })
    },

    getComment() {
      axios({
        method: "GET",
        url: "http://localhost:8080/education/comment/getComments?session_id=" + this.session_info.id,
      }).then(response => {
        let resp = response.data;
        console.log(resp.message)
        this.comments = resp.data.comments
        this.comments.forEach((comment) => {
          if (comment.father_comment_id === -1) {
            this.show_sonComments.push({comment_id: comment.id, show: false})
          }
        })
      });
    },

    postComment() {
      if (this.commentContent === '') return

      axios({
        method: "POST",
        url: "http://localhost:8080/education/comment/postComment",
        data: {
          content: this.commentContent,
          father_comment_id: -1,
          user_id: this.user_id,
          session_id: this.session_info.id,
          comment_date: dayjs().format("YYYY/MM/DD")
        },
        transformRequest: [function (data) {
          let str = "";
          for (let key in data) {
            str += encodeURIComponent(key) + "=" + encodeURIComponent(data[key]) + "&";
          }
          return str;
        }]
      })
          .then(response => {
            console.log(response.data.message);
            ElMessage({
              type: 'success',
              message: '您的评论已发布！',
            })
            this.getComment()
            this.commentContent = ''
          });
    },

    postReply(fatherComment, replyComment) {
      if (this.replyContent === '') return
      axios({
        method: "POST",
        url: "http://localhost:8080/education/comment/postReply",
        data: {
          content: this.replyContent,
          father_comment_id: fatherComment.id,
          user_id: this.user_id,
          session_id: this.session_info.id,
          comment_date: dayjs().format("YYYY/MM/DD"),
          reply_user_id: replyComment.user_id
        },
        transformRequest: [function (data) {
          let str = "";
          for (let key in data) {
            str += encodeURIComponent(key) + "=" + encodeURIComponent(data[key]) + "&";
          }
          return str;
        }]
      })
          .then(response => {
            console.log(response.data.message);
            this.getComment()
            ElMessage({
              type: 'success',
              message: '您的评论已发布！',
            })
            this.replyContent = ''
          });
    },

    show_sonComment(comment, expand) {
      for (let showSonComment of this.show_sonComments) {
        if (showSonComment.comment_id === comment.id) {
          showSonComment.show = !!expand;
        }
      }
    }
  },

  mounted() {
    this.course_id = this.$route.params.course_id
    this.user_id = sessionStorage.getItem('USER_ID')
    this.getSessionsCount()
    this.form.session = this.session_cnt + 1
    this.getSessionInfo(1)
  }
}
</script>

<style scoped>
.detail_course {
  width: 80%;
  margin: 0 auto;
  background-color: #fff;
}

.card_item {
  padding: 5px;
  height: 30px;
  font-size: 10px;
}

.card_content {

}

.student-score {
  margin-top: 10px;
  margin-bottom: 10px;
  height: 70px;
}
</style>