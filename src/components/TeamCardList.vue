<script setup lang="ts">
import {TeamType} from "../models/team";
import {teamStatusEnum} from "../constants/team";
import cat from '../assets/waterCat.png';
import myAxios from "../plugins/myAxios";
import {Dialog, Toast} from "vant";
import {onMounted, ref} from "vue";
import {getCurrent} from "../services/currentUser.ts";
import {useRouter} from "vue-router";

const showPasswordDialog = ref(false);
// const showPasswordDialog = Dialog;
const VanDialog = Dialog.Component;
const password = ref('');
const joinTeamId = ref(0);
const currentUser = ref();
const teamId = ref([])
const router = useRouter();

onMounted(async () => {
  currentUser.value = await getCurrent();
})

interface TeamCardListProps {
  teamList: TeamType[];
}

const props = withDefaults(defineProps<TeamCardListProps>(), {
  // @ts-ignore
  teamList: [] as TeamType[],
});

// console.log("props.teamList",props)

const preJoinTeam = (team: TeamType) => {
  joinTeamId.value = team.id;
  if (team.status === 0) {
    doJoinTeam()
  } else {
    showPasswordDialog.value = true;
  }
}

const doJoinCancel = () => {
  joinTeamId.value = 0;
  password.value = '';
}

/**
 * 加入队伍
 */
const doJoinTeam = async () => {
  if (!joinTeamId.value) {
    return;
  }
  const res = await myAxios.post('/team/join', {
    teamId: joinTeamId.value,
    password: password.value
  });
  if (res?.code === 0) {
    Toast.success('加入成功');
    doJoinCancel();
  } else {
    Toast.fail('加入失败' + (res.description ? `，${res.description}` : ''));
  }

}

/**
 * 跳转至更新队伍页
 * @param id
 */
const doUpdateTeam = (id: number) => {
  router.push({
    path: '/team/update',
    query: {
      id,
    }
  })
}

/**
 * 退出队伍
 * @param id
 */
const doQuitTeam = async (id: number) => {
  const res = await myAxios.post('/team/quit', {
    teamId: id
  });
  if (res?.code === 0) {
    Toast.success('操作成功');
  } else {
    Toast.fail('操作失败' + (res.description ? `，${res.description}` : ''));
  }
}

/**
 * 解散队伍
 * @param id
 */
const doDeleteTeam = async (id: number) => {
  const res = await myAxios.post('/team/delete', {
    id,
  });
  if (res?.code === 0) {
    Toast.success('操作成功');
  } else {
    Toast.fail('操作失败' + (res.description ? `，${res.description}` : ''));
  }
}

/**
 * 查看队伍
 * @param id
 */
const showTeam = (id) => {
  teamId.value.push(id)
  router.push({
    name: "teamShow",
    query: {teamId: teamId.value},
  })
}
</script>

<template>
  <div
      id="teamCardList"
  >
    <van-card
        v-for="team in props.teamList"
        :thumb="cat"
        :desc="team.description"
        :title="`${team.name}`"
    >
      <template #tags>
        <van-tag plain type="danger" style="margin-right: 8px; margin-top: 8px">
          {{
            teamStatusEnum[team.status]
          }}
        </van-tag>
        <van-tag plain style="color: #F0AF8F;margin-right: 8px; margin-top: 8px" type="primary">
          队长：{{ team.createUser.username }}
        </van-tag>
      </template>
      <template #bottom>
        <div>
          {{ `队伍人数: ${team.hasJoinNum}/${team.maxNum}` }}
        </div>
        <div v-if="team.expireTime">
          {{ '过期时间: ' + team.expireTime }}
        </div>
        <div>
          {{ '创建时间: ' + team.createTime }}
        </div>
      </template>
      <template #footer>
        <van-button size="mini" type="primary" v-if="team.userId !== currentUser?.id && !team.hasJoin" plain icon="plus" @click="preJoinTeam(team) ">
          加入队伍
        </van-button>
        <van-button v-if="team.userId === currentUser?.id" size="mini" plain
                    @click="doUpdateTeam(team.id)">更新队伍
        </van-button>
        <!-- 仅加入队伍可见 -->
        <van-button v-if="team.userId !== currentUser?.id && team.hasJoin" size="mini" plain
                    @click="doQuitTeam(team.id)">退出队伍
        </van-button>
        <van-button v-if="team.userId === currentUser?.id" size="mini" type="danger" plain
                    @click="doDeleteTeam(team.id)">解散队伍
        </van-button>
        <van-button v-if="team.hasJoin" plain size="mini" type="default" icon="eye-o" @click="showTeam(team.id)">查看队伍</van-button>
      </template>
    </van-card>
    <!--输入密码框-->
    <van-dialog v-model:show="showPasswordDialog" title="请输入队伍密码" show-cancel-button @confirm="doJoinTeam"
                @cancel="doJoinCancel">
      <van-field v-model="password" placeholder="请输入队伍密码"/>
    </van-dialog>
  </div>

</template>

<style scoped>
#teamCardList :deep(.van-image__img) {
  height: 128px;
  object-fit: unset;
}
</style>
