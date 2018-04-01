#!/usr/bin/env node
'use strict'

const { GraphQLClient } = require('graphql-request')
const options = require(process.env.HOME + '/.env')
const program = require('commander')
const token = options.GHTOKEN
const endPoint = 'https://api.github.com/graphql'
const chalk = require('chalk')
const marked = require('marked')
const TerminalRenderer = require('marked-terminal')
marked.setOptions({
  renderer: new TerminalRenderer({ reflowText: true, paragraph: chalk.green })
})

program.version('0.0.1')
  .description('get info for a github issue in automattic/mongoose')
  .option('-i, --issue <str>', 'issue to search for.')
  .option('-c, --comments', 'include issue comments in output')
  .parse(process.argv)

function craftQuery (n) {
  let cast = Number(n)
  if (Number.isNaN(cast)) {
    console.error('issue number bad')
    process.exit(1)
  } else {
    return {
      query: `query repository($n: Int!){
          repository(owner:"Automattic", name:"mongoose") {
            issue(number: $n) {
              author {
                login
              },
              bodyText,
              createdAt,
              state,
              resourcePath,
              title,
              comments(first: 100) {
                edges {
                  node {
                    author {
                      login
                    },
                    body,
                    publishedAt,
                    lastEditedAt
                  }
                }
              }
            }
          }
        }`,
      variables: {
        n: cast
      }
    }
  }
}

function sendQuery (q) {
  return new GraphQLClient(endPoint, {
    headers: {
      Authorization: `bearer ${token}`
    }
  }).request(q.query, q.variables).catch((e) => {
    console.dir(e)
    if (!e.response.data) {
      console.log('Issue number provided is probably a PR.')
      process.exit(1)
    }
  })
}

function label (str) {
  return chalk.gray.bold.underline(str)
}

function text (str) {
  let myStr = str
    .replace(/(Do you want to request a feature or report a bug\?)/,
      chalk.black('$1'))
    .replace(/(What is the current behavior\?)/,
      chalk.black('$1'))
    .replace(/(What is the expected behavior\?)/,
      chalk.black('$1'))
  return chalk.green(myStr)
}

async function run () {
  let issue = program.issue
  let query = craftQuery(issue)
  let data = await sendQuery(query)
  let o = data.repository.issue
  let author = text(o.author.login)
  let state = text(o.state)
  let created = text(o.createdAt)
  let body = text(o.bodyText)

  let details = [
    `${label('author')}: ${author}  ${label('state')}:`,
    `${state} ${label('created')} ${created}\n`
  ].join('')

  let msg = [
    `${label('title')}: ${text(o.title)}\n`,
    details,
    `${label('report')}:`,
    `${body}\n`
  ].join('\n')

  let comments = []
  o.comments.edges.forEach((c) => {
    comments.push(`${label(c.node.author.login)}: ${c.node.body}`)
    comments.push('\n')
  })

  console.log(msg)
  if (program.comments && (comments.length > 0)) {
    console.log(label('Comments') + ':')
    console.log(marked(comments.join('\n')))
  } else if (program.comments) {
    console.log(chalk.red.bold('No Comments yet.'))
  }
}

run()
