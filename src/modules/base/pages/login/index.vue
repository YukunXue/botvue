<template>
	<div class="page-login">
		<div class="box">
			<div class="logo">
				<img src="/logo.png" alt="Logo" />
				<div class="name">
					<span v-for="text in app.info.name" :key="text">{{ text }}</span>
				</div>
			</div>

			<p class="desc">ALL In Onething</p>

			<div class="op">
						<el-button type="primary" :loading="saving" @click="toLogintest"
							>{{		isWalletConnected ? '退出登录' : '连接钱包' }}</el-button
						>
			</div>

			<div v-if="!isWalletConnected">
			
			</div>
			<div class="notice_chart"  v-else>
				<p class="desc">
					欢迎您选择AIO BOT , 如果你想享受AIO BOT服务, 请及时添加官方客服
				</p>
				<!-- 在这里添加你想要显示的连接成功后的消息内容 -->
				<p class="column">
					<h1>目前 AIO BOT支持:</h1>
					<h3>撸毛工具</h3>
					<ul>
						<li>zks空投工具</li>
						<li>base空投工具</li>
					</ul>
					<h3>铭文Mint工具</h3>
					<ul>
						<li>全链Mint</li>
						<li></li>
					</ul>
					<h3>临时工具</h3>
				</p>
			</div>
		</div>

		<!-- <div class="bg">
			<cl-svg name="bg"></cl-svg>
		</div> -->
			<!-- 显示连接钱包按钮或者连接成功的消息 -->


	</div>
</template>

<script lang="ts" name="login" setup>
import { reactive, ref } from "vue";
import { ElMessage } from "element-plus";
import { useCool } from "/@/cool";
import { useBase } from "/$/base";
import { storage } from "/@/cool/utils";
import { useConnectMetamask } from "../../hooks/use-connect-metamask";
const { refs, setRefs, router, service } = useCool();
const { user, app } = useBase();
const emit = defineEmits(["update:modelValue", "change"]);

const metamask = useConnectMetamask();
// 状态
const saving = ref(false);

// 钱包连接状态
const isWalletConnected = ref(false);

const nonce = ref("");
// 表单数据
const form = reactive({
	username: "",
	password: "",
	captchaId: "",
	verifyCode: ""
});


const toLogintest = async () => {
    if (isWalletConnected.value) {
        await logout(); // 执行退出登录操作
    } else {
        await connectWallet(); // 执行连接钱包操作
    }
};

const logout = async () => {
    // 执行退出登录操作，例如清除用户信息等
	metamask.disconnect();
	isWalletConnected.value = false;
	console.log(metamask.isConnect.value)
    console.log('退出登录');
    // 这里可以添加退出登录的逻辑
};
const connectWallet = async () => {
    saving.value = true;
    try {
		await getGenNonceFromBackend();
		console.log(nonce);
        const tx = await metamask.connectMetamask(nonce.value);
		const signature = tx.signature;
		const address = tx.from[0];
		form.username = address;
		form.password = signature;
		isWalletConnected.value = true;
		console.log('钱包连接成功！');
		await toLogin();
		console.log(form.captchaId)
    } catch (error) {
        ElMessage.info('请充值以便享受后续服务：');
    } finally {
        saving.value = false;
    }
};
async function getGenNonceFromBackend() {
	await service.base.open
					.genNonce({

					}).then(({ captchaId, data }) => {
						form.captchaId = captchaId;
						console.log(data);
						nonce.value = data;
						emit("update:modelValue", captchaId);
						emit("change", {
							nonce,
							captchaId
						});
					})
					.catch((err) => {
						ElMessage.error(err.message);
					});
}
// 登录
async function toLogin() {

	saving.value = true;

	try {
		// 登录
		await service.base.open.login(form).then(user.setToken);

		// token 事件
		await Promise.all(app.events.hasToken.map((e) => e()));

		// 设置缓存
		storage.set("username", form.username);

		// 跳转首页
		router.push("/");
	} catch (err: any) {
		// 刷新验证码
		refs.picCaptcha.refresh();

		// 提示错误
		ElMessage.error(err.message);
	}

	saving.value = false;
}
</script>

<style lang="scss" scoped>
$color: #2c3142;

.page-login {
	display: flex;
	justify-content: center;
	align-items: center;
	min-height: 100vh; /* 修改为min-height */
	width: 100%;
	position: relative;
	background-color: #000;
	color: $color;
	overflow-y: auto; /* 添加垂直滚动 */
	
	.bg {
		position: absolute;
		left: 0;
		top: 0;
		height: 100%;
		width: 90%;
		pointer-events: none;
		transform: rotate(180deg) scaleY(-1);

		.cl-svg {
			height: 100%;
			width: 100%;
			fill: $color;
		}
	}

	.copyright {
		position: absolute;
		bottom: 15px;
		left: 0;
		text-align: center;
		width: 100%;
		color: #666;
		font-size: 14px;
	}

	.box {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		height: 80%;
		width: 100%;
		position: absolute;
		right: 0;
		top: 0;
		z-index: 9;

		.logo {
			height: 100px;
			margin-bottom: 20px;
			display: flex;
			align-items: center;
			color: #fff;
			img {
				height: 46px;
				background-color: $color;
				border-radius: 50px;
				border: 3px solid $color;
				margin-right: 10px;
			}

			span {
				display: inline-block;
				font-size: 38px;
				font-weight: bold;
				line-height: 1;
				letter-spacing: 3px;

				&:nth-child(6) {
					animation: dou 1s infinite linear;
				}
			}
		}

		.desc {
			font-size: 15px;
			letter-spacing: 1px;
			margin-bottom: 50px;
			color: #fff;
		}
		.notice_chart {
			position: absolute;
			left: 0;
			bottom: -400px;
			height: 40%;
			width: 90%;
			text-align: left;
		}

		.form {
			width: 300px;

			:deep(.el-form) {
				.el-form-item {
					margin-bottom: 20px;
					
				}

				.el-form-item__label {
					padding-left: 5px;
					color: #fff;
				}

				input {
					height: 45px;
					width: 100%;
					box-sizing: border-box;
					font-size: 17px;
					border: 0;
					border-radius: 0;
					background-color: #f8f8f8;
					padding: 0 15px;
					border-radius: 6px;
					position: relative;

					&:-webkit-autofill {
						box-shadow: none;
						-webkit-box-shadow: 0 0 0 1000px #f8f8f8 inset;
						box-shadow: 0 0 0 1000px #f8f8f8 inset;
					}

					&::placeholder {
						font-size: 14px;
					}
				}

				.row {
					display: flex;
					align-items: center;
					width: 100%;
					position: relative;

					.pic-captcha {
						position: absolute;
						right: 0;
						top: 0;
					}
				}
			}
		}
		.column {
			background-color: #000; // 修改为黑色背景
			border: 1px solid #ccc;
			border-radius: 5px;
			padding: 40px; // 适当调整内边距
			margin-bottom: 10px;
			color: #fff; // 文字颜色设置为白色
			left: 40px;
		}

		.column h1 {
			left: 40px;
			font-size: 36px; // 标题字号稍微调小一点
			margin-bottom: 20px; // 适当调整标题与内容的间距
		}

		.column h3 {
			left: 60px;
			font-size: 18px; // 副标题字号稍微调小一点
			margin-bottom: 20px; // 适当调整副标题与内容的间距
		}

		.column ul {
			list-style-type: none;
			padding: 0;
			margin: 0;
		}

		.column ul li {
			margin-bottom: 15px; // 调整列表项间距
		}

		.op {
			display: flex;
			justify-content: center;
			margin-top: 40px;

			:deep(.el-button) {
				height: 45px;
				width: 100%;
				font-size: 15px;
				border-radius: 6px;
				letter-spacing: 1px;
			}
		}
	}
}

@media screen and (max-width: 1024px) {
	.page-login {
		.box {
			width: 100%;
		}
	}
}
</style>
