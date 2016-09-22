# ember-run-later-helper-spike

This is a simple spike of some test helpers which make testing code with Em.run.laters easier.

Consider the [component](https://github.com/GavinJoyce/ember-run-later-helper-spike/blob/master/app/components/button-with-tasks.js):

```js
import Em from 'ember';
import { task, timeout } from 'ember-concurrency';

export default Em.Component.extend({
  buttonText: 'Click me',

  buttonPressed: task(function * () {
    this.set('buttonText', '...please wait...');
    yield timeout(1000);
    this.set('buttonText', '...continue to wait...');
    yield timeout(2000);
    this.set('buttonText', 'Click me again');
  })
});
```

With the [mock-run-later helper](https://github.com/GavinJoyce/ember-run-later-helper-spike/blob/master/tests/helpers/mock-run-later.js), these [tests](https://github.com/GavinJoyce/ember-run-later-helper-spike/blob/master/tests/integration/components/button-with-tasks-test.js) are possible:

```js
import Em from 'ember';
import hbs from 'htmlbars-inline-precompile';
import { moduleForComponent, test } from 'ember-qunit';
import { mockRunLater, restoreRunLater } from '../../helpers/mock-run-later';

moduleForComponent('button-with-tasks', { integration: true });

test('should change text after a delay (with next())', function(assert) {
  mockRunLater(this);

  assert.expect(4);

  this.render(hbs`{{button-with-tasks}}`);

  assert.equal(this.$('button').text(), 'Click me');

  Em.run(() => this.$('button').click());

  assert.equal(this.$('button').text(), '...please wait...');

  this.mockedRunLater.next();

  assert.equal(this.$('button').text(), '...continue to wait...');

  this.mockedRunLater.next();

  assert.equal(this.$('button').text(), 'Click me again');

  restoreRunLater(this);
});

test('should change text after a delay (with advanceMilliseconds())', function(assert) {
  mockRunLater(this);

  assert.expect(5);

  this.render(hbs`{{button-with-tasks}}`);

  assert.equal(this.$('button').text(), 'Click me');

  Em.run(() => this.$('button').click());

  assert.equal(this.$('button').text(), '...please wait...');

  this.mockedRunLater.advanceMilliseconds(800);

  assert.equal(this.$('button').text(), '...please wait...'); //no run.later is executed

  this.mockedRunLater.advanceMilliseconds(201); //TODO: GJ: we may be off by one here

  assert.equal(this.$('button').text(), '...continue to wait...');

  this.mockedRunLater.advanceMilliseconds(2000);

  assert.equal(this.$('button').text(), 'Click me again');

  restoreRunLater(this);
});
```
