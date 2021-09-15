To create an App Engine application in your project, execute the following command in Cloud Shell:

`gcloud app create --region "us-central"`

Note: Although You aren't yet using App Engine for your web application, Cloud Datastore requires you to create an App Engine application in your project.

```private Datastore datastore =
DatastoreOptions.getDefaultInstance().getService();

private static final String ENTITY_KIND = "Question";

private final KeyFactory keyFactory =
datastore.newKeyFactory().setKind(ENTITY_KIND);

`Key key = datastore.allocateId(keyFactory.newKey());`

Entity questionEntity = Entity.newBuilder(key)
.set(Question.QUIZ, question.getQuiz())
.set(Question.AUTHOR, question.getAuthor())
.set(Question.TITLE, question.getTitle())
.set(Question.ANSWER_ONE,question.getAnswerOne())
.set(Question.ANSWER_TWO, question.getAnswerTwo())
.set(Question.ANSWER_THREE,question.getAnswerThree())
.set(Question.ANSWER_FOUR, question.getAnswerFour())
.set(Question.CORRECT_ANSWER,
question.getCorrectAnswer())
.build();

datastore.put(questionEntity);
```
Preview  The question you just made is now in DataReturn. In the Console, click Navigation menu > Datastore > Entities to see your new question!
```
public List<Question> getAllQuestions(String quiz){

// TODO: Create the query
// The Query class has a static newEntityQueryBuilder()
// method that allows you to specify the kind(s) of
// entities to be retrieved.
// The query can be customized to filter the Question
// entities for one quiz.

Query<Entity> query = Query.newEntityQueryBuilder()
.setKind(ENTITY_KIND)
.setFilter(StructuredQuery.PropertyFilter.eq(
Question.QUIZ, quiz))
.build();

// END TODO

// TODO: Execute the query
// The datastore.run(query) method returns an iterator
// for entities

Iterator<Entity> entities = datastore.run(query);

// END TODO

// TODO: Return the transformed results
// Use the buildQuestions(entities) method to convert
// from Datastore entities to domain objects

return buildQuestions(entities);

// END TODO
}


    private List<Question> buildQuestions(Iterator<Entity> entities){
        List<Question> questions = new ArrayList<>();
        entities.forEachRemaining(entity-> questions.add(entityToQuestion(entity)));
        return questions;
    }

    private Question entityToQuestion(Entity entity){
        return new Question.Builder()
                .withQuiz(entity.getString(Question.QUIZ))
                .withAuthor(entity.getString(Question.AUTHOR))
                .withTitle(entity.getString(Question.TITLE))
                .withAnswerOne(entity.getString(Question.ANSWER_ONE))
                .withAnswerTwo(entity.getString(Question.ANSWER_TWO))
                .withAnswerThree(entity.getString(Question.ANSWER_THREE))
                .withAnswerFour(entity.getString(Question.ANSWER_FOUR))
                .withCorrectAnswer(entity.getLong(Question.CORRECT_ANSWER))
                .withId(entity.getKey().getId())
                .build();
    }


}
```

