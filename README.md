Develop chattyApp with Graphql and React-Native

Help from https://www.youtube.com/watch?v=33qP1QMmjv8


Part 1
This part is mainly for the basic setup of the server. We go over the installation of express and the basic graphql setup. We add also Mongodb and we make mocks on it.

Create a folder where we gonna put the server and mobile folder in
Cd in the server folder and run the command yarn init + click enter on each question
Run yarn add express cross-env body-parser
Create a folder call src and create a file in call index.js
Put a the top of the file the basic setup of express.
import express from 'express';
import bodyParser from 'body-parser';

const app = express(); // create an instance of express

const PORT = process.env.PORT || 3000; // create the port

app.use(bodyParser.json()); // add body-parser as the json parser middleware

app.listen(PORT, err => {
  if (err) {
    console.error(err);
  } else {
    console.log(`App listen on port: ${PORT}`);
  }
});
Now we need to install babel in the devDependencies cause we want to write in latest javascript feature.
yarn add -D babel-cli babel-plugin-transform-object-rest-spread babel-preset-env

Go in your package.json and add this scripts file:
"dev": "cross-env NODE_ENV=dev nodemon --exec babel-node src/index.js"
Create a .babelrc with this thing in.
{
  "presets": [
    [
      "env",
      {
        "targets": {
          "node": "6.10"
        }
      }
    ]
  ],
  "plugins": [
    [
      "transform-object-rest-spread",
      {
        "useBuiltIns": true
      }
    ]
  ]
}
Here I make use of babel-preset-env who help us to setup babel without a pain.

Now time to setup some graphql stuff. First create a folder graphql inside src. After create a schema.js file and put that in
export default`
  type Tweet {
    _id: String
    text: String
  }

  type Query {
    getTweets: [Tweet]
  }

  schema {
    query: Query
  }
`;
Here we have a basic schema for the tweet. Also we have a query call getTweets you render a list of this tweet.

Now time to write the resolver for this getTweets. Create a folder resolvers inside src/graphql/. After a file call tweet-resolver.js. Inside this one put this.
import Tweet from '../../models/Tweet';

export default {
  getTweets: () => Tweet.find({})
}
And create a index.js file inside src/graphql/resolvers/ folder and put

import TweetResolvers from './tweet-resolver';

export default {
  Query: {
    getTweets: TweetResolvers.getTweets
  }
}
But where this Tweet models came ? Yes this is time to setup the db.

Create a db.js inside the src/config/ folder. Put this thing after have running this command.
yarn add mongoose

/* eslint-disable no-console */

import mongoose from 'mongoose';

import constants from './constants';

mongoose.Promise = global.Promise;

mongoose.set('debug', true); // debug mode on

try {
  mongoose.connect(constants.DB_URL, {
    useMongoClient: true,
  });
} catch (err) {
  mongoose.createConnection(constants.DB_URL, {
    useMongoClient: true,
  });
}

mongoose.connection
  .once('open', () => console.log('MongoDB Running'))
  .on('error', e => {
    throw e;
  });
Nothing crazy here just the basic setup of a db with mongodb. But where do the constants file came ? Time to create it. So inside the src/config/ folder create a file call `constants.js' and put this thing.

export default {
  PORT: process.env.PORT || 3000,
  DB_URL: 'mongodb://localhost/tweeter-development',
  GRAPHQL_PATH: '/graphql'
}
Here this is the constants where we put the base configuration of the server.

PORT -> the port of the app
DB_URL -> url for the db
GRAPHQL_PATH -> the url for graphql server
Time to create the Mongodb schema of the tweet. Inside src/ folder create models folder and inside this one Tweet.js file. Inside this one we create a basic schema for now ;)
import mongoose, { Schema } from 'mongoose';

const TweetSchema = new Schema({
  text: String,
});

export default mongoose.model('Tweet', TweetSchema);
Time to update the simple express server for add graphql on it inside src/index.js
yarn add apollo-server-express graphql-tools

import express from 'express';
import { graphqlExpress, graphiqlExpress } from 'apollo-server-express';
import { makeExecutableSchema } from 'graphql-tools';
import { createServer } from 'http';
import bodyParser from 'body-parser';

import './config/db';
import constants from './config/constants';
import typeDefs from './graphql/schema';
import resolvers from './graphql/resolvers';

app.use(bodyParser.json());

app.use(
  '/graphiql',
  graphiqlExpress({
    endpointURL: constants.GRAPHQL_PATH,
  }),
);

app.use(
  constants.GRAPHQL_PATH,
  graphqlExpress({
    schema
  }),
);

graphQLServer.listen(constants.PORT, err => {
  if (err) {
    console.error(err);
  } else {
    console.log(`App running on port: ${constants.PORT}`);
  }
});
Nothing crazy here, basic setup + add graphiql the IDE of grahpql.

Time to mocks some stuff ? Sure ;) That's gonna be easy, first think add yarn add -D faker who is a library for help you have mock data. Gonna be pretty easy here we gonna create 10 fake mock tweet.
Inside src create a folder mocks and a file index.js inside this one.

import faker from 'faker';

import Tweet from '../models/Tweet';

const TWEETS_TOTAL = 10;

export default async () => {
  try {
    await Tweet.remove();

    await Array.from({ length: TWEETS_TOTAL }).forEach(async () => {
      await Tweet.create({
        text: faker.lorem.paragraphs(1),
      })
    });
  } catch (error) {
    throw error;
  }
}
Add this mocks promise to the server inside src/index.js
// all other import
import mocks from './mocks';

// other code here

mocks().then(() => {
  graphQLServer.listen(constants.PORT, err => {
    if (err) {
      console.error(err);
    } else {
      console.log(`App running on port: ${constants.PORT}`);
    }
  });
});
Time to test it. Go on https://github.com/skevy/graphiql-app and download the app. Why ? Because we gonna need this of the jwt auth later ;)

Open this tool now and add this in the left part. This is a simple query where we get all 10 tweets. Don't forget to put the url http://localhost:3000/graphql

{
  getTweets {
    _id
		text
  }
