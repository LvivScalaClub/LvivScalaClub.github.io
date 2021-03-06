#!/usr/bin/env groovy Jenkinsfile

import java.text.SimpleDateFormat

pipeline {

  agent any

  triggers {
    cron('H H 1 1 *')
  }

  parameters {
    string(
        name: 'post_title',
        description: 'Used for filename and post.sh name. The space will be replaced by `-` inv the file name.',
        defaultValue: ''
    )
    text(
        name: 'post_body',
        description: 'Use GitHub MD format',
        defaultValue: 'This week on Scala club Last week we'
    )
    booleanParam(
        name: 'post_footer',
        description: 'Add "Join us next Wednesday, at 11:00 in room"',
        defaultValue: true
    )
    choice(
        name: 'room',
        description: '',
        choices: 'Jamaika (room 223)\nSpain (room 426)'
    )
    string(
        name: 'details_url',
        description: 'Path to video or GitHub repo',
        defaultValue: ''
    )
    string(
        name: 'tags',
        description: 'Tags in lowercase are separated by commas',
        defaultValue: ''
    )
    string(
        name: 'image_url',
        description: 'Path to photo',
        defaultValue: '/images/default.jpg',
    )
  }

  environment {
    GIT_TOKEN = credentials("Jenkins-GitHub-Apps-Personal-access-tokens")
  }

  stages {

    stage('Set up env') {
      when {
        branch 'master'
      }
      steps {
        script {
          env.SKIP_AUTO_RUN = params.post_title == ''
        }
      }
    }

    stage('Set up git env') {
      when {
        branch 'master'
        environment name: 'SKIP_AUTO_RUN', value: 'false'
      }
      steps {
        script {
          dir("${env.WORKSPACE}") {
            sh 'git fetch'
            sh 'git pull'
            sh "git config remote.origin.url 'https://${env.GIT_TOKEN}@github.com/LvivScalaClub/LvivScalaClub.github.io.git'"
            sh 'git clean -fdx'
          }
        }
      }
    }

    stage('Prepare post') {
      when {
        branch 'master'
        environment name: 'SKIP_AUTO_RUN', value: 'false'
      }
      steps {
        script {
          def date = new Date()
          def dateday = new SimpleDateFormat('yyyy-MM-dd').format(date)
          def datetime = new SimpleDateFormat('yyyy-MM-dd H:m').format(date)
          def title = "${params.post_title}".replaceAll('[ #]', '-')

          env.FILENAME = "${dateday}-${title}.md"

          env.MESSAGE = "${params.post_body.replaceAll("'ll", " will")}" +
              ({params.details_url} ? "${params.details_url}" : "") +
              "\\n" +
              ({params.post_footer} ? "Join us next Wednesday, at 11:00 in ${params.room}" : "")

          env.POST = "---\n" +
              "layout: post\n" +
              "title:  \"${params.post_title}\"\n" +
              "image: ${params.image_url}\n" +
              "tags: [${params.tags}]\n" +
              "date: ${datetime}:00 +0200\n" +
              "---\n" +
              "\n" +
              "${params.post_body}" +
              ({params.details_url} ? "[${params.details_url}](${params.details_url})" : "") +
              "\n\n" +
              ({params.post_footer} ? "Join us next Wednesday, at 11:00 in ${params.room}" : "")

          sh 'printenv'
        }
      }
    }

    stage('Create post') {
      when {
        branch 'master'
        environment name: 'SKIP_AUTO_RUN', value: 'false'
      }
      steps {
        script {
          dir("${env.WORKSPACE}/_posts") {
            writeFile([file: "${env.FILENAME}", text: "${env.POST}"])
            sh 'git status'
            sh "git add ${env.FILENAME}"
            sh "git commit -m '${params.post_title}'"
            sshagent(['jenkins-deploy-keys']) {
              sh "git push origin HEAD:${env.GIT_BRANCH}"
            }
          }
        }
      }
    }
  }
}