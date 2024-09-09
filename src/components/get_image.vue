<template>



<div class="accounts">
    <div
        v-for="user in users"
        :key="user.id"
        class="account_card_wrapper"
    >
        <v-card
        class="account_card"
        router
        :to="'/account/detail/?id=' + user.id"
        >
        <div class="account_list_img_wrapper">
            <v-img
            v-if="users && user.imgSrc"
            class="account_list_img"
            :src="user.imgSrc"
            ></v-img>

            <v-img v-else class="account_list_img"></v-img>
        </div>
        <div class="account_list_items_wrapper">
            <div class="account_list_name_wrapper">
            <span>{{ user.name ? user.name : "氏名未入力" }}</span>
            </div>

            <div class="account_list_email_wrapper">
            <span>{{ user.email ? user.email : "" }}</span>
            </div>
        </div>
        <div class="account_list_chevron_wrapper">
            <div class="chevron_wrapper">
            <v-icon
                color="#FFF"
                icon="mdi-chevron-right"
                size="small"
            ></v-icon>
            </div>
        </div>
        </v-card>
    </div>
    </div>
</template>





import { getUrl } from "aws-amplify/storage";



// @ts-ignore
let users = ref<Array<any>>([]);

async function getUsers() {
  try {
    const { data: fetchedUsers, errors } = await client.models.User.list();

    fetchedUsers.forEach(async (user: any) => {
      if (user.imgPath) {
        user.imgSrc = await getImgUrl(user.imgPath);
      }

      console.log("user", user);

      users.value = [...users.value, user];
    });
  } catch (error) {
    console.log("getUsers error", error);
  }
}



// 画像URL取得
const getImgUrl = async (path: string) => {
    const linkToStorageFile = await getUrl({
      path,
      // Alternatively, path: ({identityId}) => `album/{identityId}/1.jpg`
      options: {
        validateObjectExistence: false, // defaults to false
        expiresIn: 900, // validity of the URL, in seconds. defaults to 900 (15 minutes) and maxes at 3600 (1 hour)
        useAccelerateEndpoint: false, // Whether to use accelerate endpoint.
      },
    });
    console.log("signed URL href: ", linkToStorageFile.url);
  
    return linkToStorageFile.url.toString();
  };