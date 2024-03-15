<script lang="ts">
	import PortOne from '@portone/browser-sdk/v2';
	import axios from 'axios';

	//해당 주문 정보는 백엔드에 http 요청을 보내서 조회해야 함.
	//예시 코드에서는 하드코딩되어 있음.

	const orderName = '라이브러리 오브 루이나';
	const currency = 'CURRENCY_KRW';
	const totalAmount = 1000;
	const fullName = '홍길동';
	const email = 'honggildong@example.com';

	async function requestKakaoPayment() {
		const response = await PortOne.requestPayment({
			storeId: import.meta.env.VITE_STORE_ID,
			channelKey: import.meta.env.VITE_CHANNEL_KEY,
			paymentId: `payment-${crypto.randomUUID()}`,
			orderName,
			isTestChannel: true,
			totalAmount,
			currency,
			payMethod: 'EASY_PAY',
			customer: {
				fullName,
				email
			}
		});
		if (response?.code != null) {
			alert(`결제 요청 실패: ${response.message}`);
			return;
		} else {
			const payment = await checkPayment({
				paymentId: response?.paymentId as string,
				order: {
					orderName,
					currency,
					payMethod: 'EASY_PAY',
					amount: totalAmount
				}
			});

			if (payment.status == 'OK') {
				alert('결제 성공!');
			}
		}
	}
	async function requestCardPayment() {
		const response = await PortOne.requestPayment({
			storeId: import.meta.env.VITE_STORE_ID,
			// 채널 키 설정
			channelKey: import.meta.env.VITE_CHANNEL_KEY,
			paymentId: `payment-${crypto.randomUUID()}`,
			orderName,
			isTestChannel: true,
			totalAmount,
			currency,
			payMethod: 'CARD',
			customer: {
				fullName,
				email
			}
		});

		if (response?.code != null) {
			alert(`결제 요청 실패: ${response.message}`);
			return;
		} else {
			console.log(response);
			const payment = await checkPayment({
				paymentId: response?.paymentId as string,
				order: {
					orderName,
					currency,
					payMethod: 'CARD',
					amount: totalAmount
				}
			});

			if (payment.status == 'OK') {
				alert('결제 성공!');
			}
		}
	}

	async function checkPayment(paymentInfo: PaymentDTO) {
		const response = await axios.post('http://localhost:3000/payment/complete', paymentInfo);
		return response.data;
	}

	class PaymentDTO {
		constructor(
			paymentId: string,
			order: {
				orderName: string;
				currency: string;
				payMethod: string;
				amount: number;
			}
		) {
			this.paymentId = paymentId;
			this.order = order;
		}
		paymentId: string;
		order: {
			orderName: string;
			currency: string;
			payMethod: string;
			amount: number;
		};
	}
</script>

<section>
	<button on:click={requestCardPayment}>신용카드 결제 요청</button>
	<button on:click={requestKakaoPayment}>카카오페이 결제 요청</button>
</section>
