<template>
	<div class="date">
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
				<p @click="addLastYear">上一年</p>
				<div v-for="(item, index) in render">
					<p class="name">{{ item.name }}</p>
					<div class="data">
						<p v-for="(item, index) in item.data" :class="[ item.status ]" @click="click(item, index)">{{ item.date }}</p>
					</div>
				</div>
				<p @click="addNextYear">下一年</p>
			</div>

		</div>
	</div>
</template>

<script>

export default {
	name: 'date',
	data() {
		return {

			render: [],
			years: [],

			status: {
				normal: "normal",
				start: "start",
				end: "end",
				continuous: "continuous",
				disable: "disable"
			},

			selected: {
				startIndex: -1,
				endIndex: -1
			},

		}
	},
	computed: {

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
	components: {

	},

	methods: {
		parsePropStartEnd () {
			let start = parse(this.start);
			let end = parse(this.end);

			if(start[0] >= end[0]){
				this.years.push(start[0]);
			}else{
				for(let i = start[0]; i <= end[0]; i++){
					this.years.push(i);
				}
			}

			function parse (val) {
				let sep = "";

				if (val) {
					if (val.indexOf("-") !== -1) sep = "-";
					if (val.indexOf(".") !== -1) sep = ".";
					if (val.indexOf("/") !== -1) sep = "/";

					let split = val.split(sep);

					return [parseInt(split[0]), parseInt(split[1]) - 1, parseInt(split[2])];
				}else{
					let now = new Date();
					return [now.getFullYear(), now.getMonth(), now.getDate()];
				}
			}
		},

		getRenderMonth (y, m) {
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
		getRenderYear (y) {
			let render = [];

			for (let i = 0; i < 12; i++){
				render.push({
					name: y + " - " + zero(i + 1),
					data: this.getRenderMonth(y, i)
				});
			}

			function zero(n){
				return n < 10 ? '0' + n : n;
			}

			return render;
		},
		initRender () {
			for(let i = 0; i < this.years.length; i++){
				this.render.push(...this.getRenderYear(this.years[i]));
			}
		},

		click (item, index) {
			switch (item.status) {
				case "disable":
					return false;

				case "start":
					this.render[index].status = "";

					if(this.selected.endIndex !== -1){
						this.clearContinuous(this.selected.startIndex, this.selected.endIndex);

						this.render[this.selected.endIndex].status = this.status.start;
						this.selected.startIndex = this.selected.endIndex;
						this.selected.endIndex = -1;

						return;
					}

					this.selected.startIndex = -1;

					break;

				case "end":
					this.clearContinuous(this.selected.startIndex, this.selected.endIndex);

					this.render[index].status = "";
					this.selected.endIndex = -1;

					break;

				default:
					if(this.selected.startIndex === -1){
						this.selected.startIndex = index;
						this.render[index].status = this.status.start;
					}else if(this.selected.endIndex === -1){
						if(index < this.selected.startIndex){
							this.selected.endIndex = this.selected.startIndex;
							this.selected.startIndex = index;

							this.render[this.selected.startIndex].status = this.status.start;
							this.render[this.selected.endIndex].status = this.status.end;

							return;
						}

						this.selected.endIndex = index;
						this.render[index].status = this.status.end;
					}else{
						this.render[this.selected.startIndex].status = "";
						this.render[this.selected.endIndex].status = "";

						this.clearContinuous(this.selected.startIndex, this.selected.endIndex);

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
		},

		setContinuous (start, end){
			for(let i = start + 1; i < end; i++){
				this.render[i].status = this.status.continuous;
			}
		},
		clearContinuous (start, end){
			for(let i = start + 1; i < end; i++){
				this.render[i].status = "";
			}
		},

		addLastYear () {
			let n = this.years[0] - 1;

			this.years.unshift(n);
			this.render.unshift(...this.getRenderYear(n));
		},
		addNextYear () {
			let n = this.years[this.years.length - 1] + 1;

			this.years.unshift(n);
			this.render.push(...this.getRenderYear(n));
		}
	},

	mounted() {
		this.parsePropStartEnd();

		this.initRender();

		// deep watch
		this.$watch("selected", function(val){
			let start = val.startIndex;
			let end = val.endIndex;

			if(start !== -1 && end !== -1 && start !== end){
				this.setContinuous(start, end);
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
	overflow-y: auto;

	&-head{
		position: fixed;
		top: 0;
		left: 0;

		width: 100%;
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
			position: fixed;
			top: 5rem;
			left: 0;

			display: flex;
			justify-content: space-around;

			width: 100%;
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
			margin: 10rem 0 2rem;

			.name{
				margin: 3rem 0 0 3%;

				color: #666;
			}

			.data{
				display: flex;
				justify-content: space-around;
				flex-wrap: wrap;

				font-size: 2rem;
				color: #333;
				text-align: center;
				line-height: 14vw;

				box-sizing: border-box;

				p{
					width: 14vw;
					height: 14vw;

					margin: 0 0 1.5% 0;

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
						color: #f6f6f6;
					}
				}
			}
		}
	}
}

</style>

