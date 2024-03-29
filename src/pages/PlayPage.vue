<script lang="ts" setup>
import { reactive } from "vue";
import { Tetromino, TETROMINO_TYPE } from '../common/Tetromino';
import { Field } from '../common/Field';

import TetrominoPreviewComponent from '../components/TetrominoPreviewComponent.vue';

let staticField = new Field();
const tetris = reactive({
  field: new Field(),
});

const tetromino = reactive({
  current: Tetromino.random(),
  position: { x: 3, y: 0 },
  next: Tetromino.random(),
});

const classBlockColor = (_x: number, _y: number): string => {
  const type = tetris.field.data[_y][_x];

  // テトリスのフィールドのマス目が空白以外なら、テトリミノの識別子を返却する
  if(type > 0) {
    return Tetromino.id(type as TETROMINO_TYPE);
  }

  // テトリスのフィールドのマス目が空白であれば、
  // 落下中のテトリミノの描画範囲をチェックする
  const { x, y } = tetromino.position;
  const { data } = tetromino.current;

  if (y <= _y && _y < y + data.length) {
    const cols = data[_y - y];
    if (x <= _x && _x < x + cols.length) {
      if (cols[_x - x] > 0) {
        return Tetromino.id(cols[_x - x] as TETROMINO_TYPE);
      }
    }
  }

  // 上記いずれの条件にも当てはまらければ、空白を返却する
  return "";
}

const canDropCurrentTetromino = (): boolean => {
  const { x, y } = tetromino.position;
  const droppedPosition = {x, y: y + 1};

  const data = tetromino.current.data;
  return tetris.field.canMove(data, droppedPosition);
}

const nextTetrisField = () => {
  const data = tetromino.current.data;
  const position = tetromino.position;

  tetris.field.update(data, position);

  staticField = new Field(tetris.field.data);
  tetris.field = Field.deepCopy(staticField);

  tetromino.current = tetromino.next;
  tetromino.next = Tetromino.random();
  tetromino.position = { x: 3, y: 0 };
}

setInterval(() => {
  tetris.field = Field.deepCopy(staticField);

  if(canDropCurrentTetromino()) {
    tetromino.position.y++;
  } else {
    nextTetrisField();
  }
}, 1 * 1000);

tetris.field.update(tetromino.current.data, tetromino.position);

</script>

<template>
  <h1>プレイ画面</h1>
  <p>ユーザー名：{{ $route.query.name }}</p>

  <div class="container">
    <div class="tetris">
      <table class="field" style="border-collapse: collapse">
        <tr v-for="(row, y) in tetris.field.data" :key="y">
          <td
              class="block"
              v-for="(col, x) in row"
              :key="() => `${x}${y}`"
              v-bind:class="classBlockColor(x, y)"
          />
        </tr>
      </table>
    </div>
    <div class="information">
      <TetrominoPreviewComponent v-bind:tetromino="tetromino.next.data"/>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.container {
  display: flex;
  justify-content: center;
  align-items: stretch;
}

.field {
  border: ridge 0.4em #2c3e50;
  border-top: none;
}

.block {
  width: 1em;
  height: 1em;
  border: 0.1px solid #95a5a6;

  &-i {
    background: #3498db;
  }
  &-o {
    background: #f1c40f;
  }
  &-t {
    background: #9b59b6;
  }
  &-j {
    background: #1e3799;
  }
  &-l {
    background: #e67e22;
  }
  &-s {
    background: #2ecc71;
  }
  &-z {
    background: #e74c3c;
  }
}

.information {
  margin-left: 0.5em;
}
</style>