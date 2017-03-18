<template>
	<div class="date" v-show="show">
		<div class="date-head">
			<svg @click="close" class="date-head-cancel" viewBox="0 0 16 16" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"><g class="transform-group"><g transform="scale(0.015625, 0.015625)"><path d="M671.968 912c-12.288 0-24.576-4.672-33.952-14.048L286.048 545.984c-18.752-18.72-18.752-49.12 0-67.872l351.968-352c18.752-18.752 49.12-18.752 67.872 0 18.752 18.72 18.752 49.12 0 67.872l-318.016 318.048 318.016 318.016c18.752 18.752 18.752 49.12 0 67.872C696.544 907.328 684.256 912 671.968 912z" fill="#fff"></path></g></g></svg>
			<p @click="submit" class="date-head-ok">确定</p>
		</div>
		<div class="date-calendar">
			<div class="date-calendar-head">
				<p>日</p>
				<p>一</p>
				<p>二</p>
				<p>三</p>
				<p>四</p>
				<p>五</p>
				<p>六</p>
			</div>
			<div class="date-calendar-body">
				<p v-for="(item, index) in render" :class="[ item.status ]" @click="click(item, index)">{{ item.date }}</p>
			</div>
		</div>
	</div>
</template>

<script>

export default {
	name: 'date',
	data() {
		return {

			// render: [
			// 	{
			// 		date: 18,
			// 		dateObj: "new Date(2017, 03, 18)",
			// 		status: "start"
			// 	}
			// ]
			render: [],

			status: {
				start: "start",
				end: "end",
				continuous: "continuous",
				disable: "disable"
			},

			selected: {
				startIndex: -1,
				endIndex: -1
			}
		}
	},
	props: {
		show: {
			type: Boolean,
			default: false
		},
		start: {
			type: String
		},
		end: {
			type: String
		}
	},
	methods: {
		initYearMonth (y, m) {
			let a = [];
			if( typeof y !== "number" ||
				typeof m !== "number" ||
				y < 0 ||
				m < 0 ||
				m > 11 ){

				let now = new Date();
				a = [now.getFullYear(), now.getMonth()];
			}else{
				a = [y, m];
			}

			return a;
		},
		getRenderArr (y, m) {
			let render = [];

			let lastDateOfMonth = new Date(y, m + 1, 0).getDate();
			for(let i = 1; i <= lastDateOfMonth; i++) {
				render.push({
					date: i,
					dateObj: new Date(y, m, i),
					status: ""
				});
			}

			let lastDateObjOfLastMonth = new Date(y, m, 0);
			let lastDayOfLastMonth = lastDateObjOfLastMonth.getDay();
			if(lastDayOfLastMonth !== 6){
				let lastDateOfLastMonth = lastDateObjOfLastMonth.getDate();
				for(let i = 0; i <= lastDayOfLastMonth; i++){
					render.unshift({
						date: lastDateOfLastMonth - i,
						dateObj: "",
						status: this.status.disable
					});
				}
			}

			let lastDayOfMonth = new Date(y, m + 1, 0).getDay();
			if(lastDayOfMonth !== 6){
				for(let i = lastDayOfMonth, j = 1; i < 6; i++, j++){
					render.push({
						date: j,
						dateObj: "",
						status: this.status.disable
					});
				}
			}

			return render;
		},
		click (item, index) {
			switch (item.status) {
				case "disable":
					return false;

				case "start":
					this.render[index].status = "";
					this.selected.startIndex = -1;
					break;

				case "end":
					this.render[index].status = "";
					this.selected.endIndex = -1;
					break;

				default:
					if(this.selected.startIndex === -1){
						this.selected.startIndex = index;
						this.render[index].status = this.status.start;
					}else if(this.selected.endIndex === -1){
						if(index <= this.selected.startIndex){
							return false;
						}

						this.selected.endIndex = index;
						this.render[index].status = this.status.end;
					}else{
						this.render[this.selected.startIndex].status = "";
						this.render[this.selected.endIndex].status = "";

						for(let i = this.selected.startIndex + 1; i < this.selected.endIndex; i++){
							this.render[i].status = "";
						}

						this.selected.startIndex = index;
						this.selected.endIndex = -1;
						this.render[index].status = this.status.start;
					}
			}
		},
		submit () {
			this.$emit("daterange", {
				start: this.render[this.selected.startIndex].dateObj,
				end: this.render[this.selected.endIndex].dateObj
			});

			this.close();
		},
		close () {
			this.show = false;
		}
	},
	mounted() {

		// render calendar
		this.render = this.getRenderArr(...this.initYearMonth());

		// deep watch
		this.$watch("selected", function(val){
			let start = val.startIndex;
			let end = val.endIndex;

			if(start !== -1 && end !== -1){
				for(let i = start + 1; i < end; i++){
					this.render[i].status = this.status.continuous;
				}
			}
		}, {
			deep: true
		});
	}
}
</script>

<style scoped lang="less">

.date{
	position: fixed;
	top: 0;
	left: 0;
	z-index: 10;

	height: 100%;
	width: 100%;

	background-color: #f6f6f6;

	&-head{
		height: 5rem;

		padding: 1rem 1.7rem 1rem 1rem;

		font-size: 2rem;
		line-height: 3.2rem;
		color: #fff;

		background-color: #39f;
		box-sizing: border-box;
		overflow: hidden;

		&-cancel{
			display: inline-block;

			width: 3rem;
			height: 3rem;
		}

		&-ok{
			float: right;
		}
	}

	&-calendar{

		&-head{
			display: flex;
			justify-content: space-around;

			height: 5rem;
			padding: 1rem 0;

			font-size: 2rem;
			line-height: 3rem;
			color: #fff;
			text-align: center;

			background-color: #39f;
			box-sizing: border-box;

			p{
				width: 14%;
			}
		}

		&-body{
			display: flex;
			justify-content: space-around;
			flex-wrap: wrap;

			padding-top: 3%;

			font-size: 2rem;
			color: #333;
			text-align: center;
			line-height: 14vw;

			box-sizing: border-box;

			p{
				width: 14vw;
				height: 14vw;

				margin: 1.5% 0;

				&.start{
					color: #fff;
					background-color: #39f;

					border-radius: 50%;
				}

				&.end{
					color: #fff;
					background-color: #fa3;

					border-radius: 50%;
				}

				&.continuous{
					background-color: #bbb;

					border-radius: 50%;
				}

				&.disable{
					color: #bbb
				}
			}
		}
	}
}

</style>

