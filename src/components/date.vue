<template>
	<div class="date" v-if="show">
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
				<p @click="addLastMonth" class="btn-month">上一月</p>
				<div v-for="(item, i1) in render">
					<p class="name">{{ item.name }}</p>
					<div class="data">
						<p v-for="(item, i2) in item.data" :class="[ item.status ]" @click="click(item, i1, i2)">{{ item.date }}</p>
					</div>
				</div>
				<p @click="addNextMonth" class="btn-month">下一月</p>
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
			months: [],

			status: {
				normal: "normal",
				start: "start",
				end: "end",
				continuous: "continuous",
				disable: "disable"
			},

			selected: {
				startIndex: [-1, -1],
				endIndex: [-1, -1]
			},

			show: true

		}
	},
	computed: {

	},
	props: {
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

			if(start[0] > end[0]){
				this.months.push([start[0], start[1]]);
			}else if(start[0] === end[0]){
				for(let i = start[1]; i <= end[1]; i++){
					this.months.push([start[0], i]);
				}
			}else{
				for(let i = start[1]; i < 12; i++){
					this.months.push([start[0], i]);
				}
				for(let i = start[0] + 1; i < end[0]; i++){
					for(let j = 0; j < 12; j++){
						this.months.push([i, j]);
					}
				}
				for(let i = 0; i < end[1]; i++){
					this.months.push([end[0], i]);
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
					status: this.status.normal
				});
			}

			let lastDateObjOfLastMonth = new Date(y, m, 0);
			let lastDayOfLastMonth = lastDateObjOfLastMonth.getDay();
			if(lastDayOfLastMonth !== 6){
				let lastDateOfLastMonth = lastDateObjOfLastMonth.getDate();
				for(let i = 0; i <= lastDayOfLastMonth; i++){
					render.unshift({
						// date: lastDateOfLastMonth - i,
						date: "",
						dateObj: "",
						status: this.status.disable
					});
				}
			}

			let lastDayOfMonth = new Date(y, m + 1, 0).getDay();
			if(lastDayOfMonth !== 6){
				for(let i = lastDayOfMonth, j = 1; i < 6; i++, j++){
					render.push({
						// date: j,
						date: "",
						dateObj: "",
						status: this.status.disable
					});
				}
			}

			return {
				y,
				m,
				name: y + " - " + zero(m + 1),
				data: render
			};

			function zero(n) {
	            return n < 10 ? '0' + n : n
	        }
		},
		initRender () {
			for(let i = 0; i < this.months.length; i++){
				this.render.push(this.getRenderMonth(...this.months[i]));
			}
		},

		click (item, i1, i2) {
			switch (item.status) {
				case this.status.disable:
					return false;

				case this.status.start:
					this.clearContinuous(this.selected.startIndex, this.selected.endIndex);

					if(this.selected.endIndex[0] !== -1 && this.selected.endIndex[1] !== -1){
						let temp = this.selected.endIndex;

						this.clearStart(...this.selected.startIndex);
						this.clearEnd(...this.selected.endIndex);
						this.setStart(...temp);

						return;
					}
					break;

				case this.status.end:
					this.clearContinuous(this.selected.startIndex, this.selected.endIndex);

					this.clearEnd(i1, i2);

					break;

				default:
					if(this.selected.startIndex[0] === -1 && this.selected.startIndex[1] === -1){
						this.setStart(i1, i2);
					}else if(this.selected.endIndex[0] === -1 && this.selected.endIndex[1] === -1){
						if(i1 < this.selected.startIndex[0] || ( i1 === this.selected.startIndex[0] && i2 <= this.selected.startIndex[1]) ){
							this.setEnd(...this.selected.startIndex);
							this.setStart(i1, i2);
							this.setContinuous([i1, i2], this.selected.endIndex);

							return;
						}

						this.setEnd(i1, i2);
						this.setContinuous(this.selected.startIndex, this.selected.endIndex);

					}else{
						this.clearContinuous(this.selected.startIndex, this.selected.endIndex);

						this.clearEnd(...this.selected.endIndex);
						this.clearStart(...this.selected.startIndex);
						this.setStart(i1, i2);
					}
			}
		},
		setStart (i1, i2) {
			this.selected.startIndex = [i1, i2];
			this.render[i1].data[i2].status = this.status.start;
		},
		clearStart (i1, i2) {
			this.selected.startIndex = [-1, -1];
			this.render[i1].data[i2].status = this.status.normal;
		},
		setEnd (i1, i2) {
			this.selected.endIndex = [i1, i2];
			this.render[i1].data[i2].status = this.status.end;
		},
		clearEnd (i1, i2) {
			this.selected.endIndex = [-1, -1];
			this.render[i1].data[i2].status = this.status.normal;
		},

		submit () {
			this.$emit("daterange", {
				start: this.render[this.selected.startIndex[0]].data[this.selected.startIndex[1]].dateObj,
				end: this.render[this.selected.endIndex[0]].data[this.selected.endIndex[1]].dateObj
			});

			this.close();
		},
		close () {
			this.show = false;
		},

		setContinuous (start, end){
			// it will cover the item which has "disabled" status
			let item = {};
			if(start[0] === end[0]){
				for(let i = start[1] + 1; i < end[1]; i++){
					item = this.render[start[0]].data[i];
					if(item.status !== this.status.disable){
						item.status = this.status.continuous;
					}
				}
			}else if(end[0] > start[0]){
				for(let i = start[1] + 1; i < this.render[start[0]].data.length; i++){
					item = this.render[start[0]].data[i];
					if(item.status !== this.status.disable){
						item.status = this.status.continuous;
					}
				}

				for(let i = start[0] + 1; i < end[0]; i++){
					for(let j = 0; j < this.render[i].data.length; j++){
						item = this.render[i].data[j];
						if(item.status !== this.status.disable){
							item.status = this.status.continuous;
						}
					}
				}

				for(let i = 0; i < end[1]; i++){
					item = this.render[end[0]].data[i];
					if(item.status !== this.status.disable){
						item.status = this.status.continuous;
					}
				}
			}
		},
		clearContinuous (start, end){
			// it will cover the item which has "disabled" status
			let item = {};
			if(start[0] === end[0]){
				for(let i = start[1] + 1; i < end[1]; i++){
					item = this.render[start[0]].data[i];
					if(item.status !== this.status.disable){
						item.status = this.status.normal;
					}
				}
			}else if(end[0] > start[0]){
				for(let i = start[1] + 1; i < this.render[start[0]].data.length; i++){
					item = this.render[start[0]].data[i];
					if(item.status !== this.status.disable){
						item.status = this.status.normal;
					}
				}

				for(let i = start[0] + 1; i < end[0]; i++){
					for(let j = 0; j < this.render[i].data.length; j++){
						item = this.render[i].data[j];
						if(item.status !== this.status.disable){
							item.status = this.status.normal;
						}
					}
				}

				for(let i = 0; i < end[1]; i++){
					item = this.render[end[0]].data[i];
					if(item.status !== this.status.disable){
						item.status = this.status.normal;
					}
				}
			}
		},

		addLastMonth () {
			let y = this.months[0][0];
			let m = this.months[0][1];

			let a = [];

			if(m === 0){
				a = [y - 1, 11];
			}else{
				a = [y, m-1];
			}

			this.months.unshift(a);
			this.render.unshift(this.getRenderMonth(...a));

			this.selected.startIndex[0]++;
			this.selected.endIndex[0]++;
		},
		addNextMonth () {
			let y = this.months[this.months.length - 1][0];
			let m = this.months[this.months.length - 1][1];

			let a = [];

			if(m === 11){
				a = [y + 1, 0];
			}else{
				a = [y, m + 1];
			}

			this.months.push(a);
			this.render.push(this.getRenderMonth(...a));

			this.selected.startIndex[0]--;
			this.selected.endIndex[0]--;
		}
	},

	mounted() {
		this.parsePropStartEnd();

		this.initRender();
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
			// fix less calc and css3 calc
			height: calc(~"100vh - 10rem");
			overflow-y: auto;

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

			.btn-month{
				padding: 1rem 0;

				color: #999;
				text-align: center;
			}
		}
	}
}

</style>

