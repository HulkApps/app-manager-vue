<template>
    <PSkeletonPage title="Plans"
                   :fullWidth="false"
                   primaryAction
                   :secondaryActions="2"
                   :breadcrumbs="false"
                   v-if="planLoading">
        <PLayout>
            <PLayoutSection oneThird="">
                <PCard sectioned="">
                    <PTextContainer>
                        <PSkeletonDisplayText size="small" />
                        <PSkeletonBodyText />
                    </PTextContainer>
                </PCard>
            </PLayoutSection>
            <PLayoutSection oneThird="">
                <PCard sectioned="">
                    <PTextContainer>
                        <PSkeletonDisplayText size="small" />
                        <PSkeletonBodyText />
                    </PTextContainer>
                </PCard>
            </PLayoutSection>
            <PLayoutSection oneThird="">
                <PCard sectioned="">
                    <PTextContainer>
                        <PSkeletonDisplayText size="small" />
                        <PSkeletonBodyText />
                    </PTextContainer>
                </PCard>
            </PLayoutSection>
        </PLayout>
        <PLayout style="margin-top: 20px">
            <PLayoutSection>
                <PCard sectioned=""><PSkeletonBodyText /></PCard>
                <PCard sectioned=""><PSkeletonBodyText /></PCard>
                <PCard sectioned=""><PSkeletonBodyText /></PCard>
                <PCard sectioned=""><PSkeletonBodyText /></PCard>
            </PLayoutSection>
        </PLayout>
    </PSkeletonPage>
    <PEmptyState
            heading="No Plans"
            image="https://cdn.shopify.com/s/files/1/0262/4071/2726/files/emptystate-files.png"
            v-else-if="!this.planLoading && this.plans.length === 0"
    >
    </PEmptyState>
    <PPage
            class="app-manager-plan-page-slider custom-title"
            title="Choose plan"
            :subtitle = "subtitleContent"
            v-else
    >

        <PStack slot="primaryAction">
            <PStackItem style="margin-top: 20px">
                <PButtonGroup class="btn-group" segmented>
                    <PButton v-if="monthlyPlan.length && yearlyPlan.length" :style="selectedPlan === 'monthly' ? monthlySelectedStyle : monthlyStyle "  @click="selectPlan('monthly')">
                        <p style="font-size: 17px; font-weight: 500" slot="default">{{('Monthly')}}</p>
                    </PButton>

                    <PButton v-if="yearlyPlan.length && monthlyPlan.length" :style="selectedPlan === 'annually' ? yearlySelectedStyle : yearlyStyle " @click="selectPlan('annually')" :primary="selectedPlan === 'annually' " >
                        <YearlyPlanPromotion />
                    </PButton>
                </PButtonGroup>
            </PStackItem>
        </PStack>
        <hr style="width: 100%; margin-right: auto;margin-left: auto;margin-bottom: 20px;" />
        <!--=======================================================-->

        <PLayout class="custom-plan">
            <PLayoutSection style="display: flex;border-radius: 20px;">
                <template style="margin-bottom: 20px; display: flex !important;">
                    <template >
                        <div class="Polaris-ResourceList__ResourceListWrapper features" style="width: 30%">
                            <div class="plan__price"></div>
                            <ul class="Polaris-ResourceList">
                                <li class="Polaris-ResourceList__ItemWrapper pro_title" :class="`${feature.value_type}__type__${feature.slug} feature__type__${feature.value_type} feature__class`" v-for="(feature, key) in features" :key="key">
                                    <div class="Polaris-ResourceList-Item__Container">
                                        <div class="Polaris-ResourceList-Item__Content">
                                            <h1 class="for-price-per-month"><span>{{ feature.name }}</span></h1>
                                        </div>
                                    </div>
                                </li>
                            </ul>
                        </div>
                    </template>
                    <carousel style="width: 70%" :per-page="perPage" :mouseDrag="false" :navigation-enabled="true" :paginationEnabled="false" :navigateTo="[this.currentSlide,true]" @transition-start="handleNavigationClick($event)">
                        <template>
                            <slide :id="key" :class="`slide-${key}`" v-for="(plan, key) in selectedPlan === 'monthly' ? monthlyPlan : yearlyPlan" :key="`slide-${key}`" >
                                <div class="plan__price" :style="activePlanStyle(plan)">
                                    <div v-if="plan.price === 0">
                                        <b v-if="plan.name !== 'Free'" style="font-size: 16px">{{(plan.name)}}</b>
                                        <p style="display: flex;margin-top: 10px">
                                            <PHeading style="font-size: 25px;font-weight: 700;">Free</PHeading>
                                        </p>
                                    </div>
                                    <div v-else-if="plan.discount && plan.discount > 0" >
                                        <b style="font-size: 16px">{{(plan.name)}}</b>
                                        <p style="display: flex;margin-top: 10px">
                                            <PHeading style="font-size: 25px;font-weight: 700;">${{parseFloat(calculateDiscountedPrice(plan)).toFixed(2)}}</PHeading>
                                            <b style="margin-top: 5px;font-size: 17px">/{{selectedPlan === 'monthly' ? ("mo") : ("year")}}</b>
                                        </p>
                                        <p style="display: flex;margin-top: 7px">
                                            <PHeading style="font-size: 18px;font-weight: 500; text-decoration:line-through;">${{parseFloat(plan.price).toFixed(2)}}</PHeading>
                                            <b style="margin-top: 3px;font-size: 14px">/{{selectedPlan === 'monthly' ? ("mo") : ("year")}}</b>
                                        </p>
                                    </div>
                                    <div v-else>
                                        <b style="font-size: 16px">{{(plan.name)}}</b>
                                        <p style="display: flex;margin-top: 10px">
                                            <PHeading style="font-size: 25px;font-weight: 700;">${{parseFloat(plan.price).toFixed(2)}}</PHeading>
                                               <b style="margin-top: 5px;font-size: 17px">/{{selectedPlan === 'monthly' ? ("mo") : ("year")}}</b>
                                        </p>
                                    </div>
                                </div>
                                <div>
                                    <ul>
                                        <li v-for="(feature, key) in features" :class="`${feature.value_type}__type__${feature.slug} feature__list feature__type__${feature.value_type}`" :key="key" :style="activePlanStyle(plan)">
                                            <div>
                                                <template v-if="plan.features && plan.features[feature.uuid]" style="display: flex">
                                                    <template v-if="plan.features[feature.uuid].value_type === 'boolean'">
                                                        <PIcon v-if="parseInt(plan.features[feature.uuid].value) === 1" color="success" source="TickMinor"/>
                                                        <PIcon v-else color="subdued" source="MinusMinor"/>
                                                    </template>
                                                    <template v-else style="display: flex">
                                                        <span v-if="plan.features[feature.uuid].value">{{ format(plan.features[feature.uuid]) }}</span>
                                                        <PIcon v-else color="subdued" source="MinusMinor"/>
                                                    </template>
                                                </template>
                                                <template v-else>
                                                    <PIcon color="subdued" source="MinusMinor"/>
                                                </template>
                                            </div>
                                        </li>
                                        <li>
                                            <PButton v-if="isCurrentPlan(plan)" :disabled="isCurrentPlan(plan)"
                                                     full-width
                                                     :pressed="isCurrentPlan(plan)">
                                                {{ ('Current Plan') }}
                                            </PButton>
                                            <PButton v-else-if="!plan.store_base_plan || plan.shopify_plans.includes(shop.shopify_plan)"
                                                     full-width
                                                     @click="plan ? getPlanUrl(plan):'javascript:void'"
                                                     :primary="true" >
                                                {{ ('Choose Plan') }}
                                            </PButton>
                                            <PButton v-else :disabled="true"
                                                     full-width
                                                     :pressed="true">
                                                {{ ('Not applicable') }}
                                            </PButton>
                                        </li>
                                    </ul>
                                </div>
                            </slide>
                        </template>
                    </carousel>
                </template>
            </PLayoutSection>
        </PLayout>
        <!--====================================================================-->
        <PStack v-if="onboard" class="choose-plan-btn" alignment="center" distribution="center" vertical>
            <PStackItem fill>
                <PButton plain @click="activePlan">{{ ('I will choose the plan later') }}</PButton>
            </PStackItem>
        </PStack>
        <PlanBanners />
    </PPage>
</template>

<script>

    import axios from "axios";
    import PlanBanners from "./PlanBanners";
    import YearlyPlanPromotion from "./YearlyPlanPromotion";
    import PPage from "../polaris-vue/src/components/PPage/PPage";
    import PStack from "../polaris-vue/src/components/PStack/PStack";
    import PStackItem from "../polaris-vue/src/components/PStack/components/PStackItem/PStackItem";
    import {PButton} from "../polaris-vue/src/components/PButton";
    import {PButtonGroup} from "../polaris-vue/src/components/PButtonGroup";
    import {PHeading} from "../polaris-vue/src/components/PHeading";
    import {PLayout} from "../polaris-vue/src/components/PLayout";
    import {PLayoutSection} from "../polaris-vue/src/components/PLayout/components/PLayoutSection";
    import {PTextContainer} from "../polaris-vue/src/components/PTextContainer";
    import {PDataTable} from "../polaris-vue/src/components/PDataTable";
    import {PDataTableCol} from "../polaris-vue/src/components/PDataTable/components/PDataTableCol";
    import {PDataTableRow} from "../polaris-vue/src/components/PDataTable/components/PDataTableRow";
    import {PIcon} from "../polaris-vue/src/components/PIcon";
    import {PTextStyle} from "../polaris-vue/src/components/PTextStyle";
    import {PCard} from "../polaris-vue/src/components/PCard"
    import {PCardSection} from "../polaris-vue/src/components/PCard/components/PCardSection"
    import {PSkeletonPage} from "../polaris-vue/src/components/PSkeletonPage"
    import {PSkeletonDisplayText} from "../polaris-vue/src/components/PSkeletonDisplayText"
    import {PSkeletonBodyText} from "../polaris-vue/src/components/PSkeletonBodyText"
    import {PEmptyState} from "../polaris-vue/src/components/PEmptyState"
    import {Carousel, Slide} from 'vue-carousel';

    export default {
        name: "AppManagerSliderPlan",
        components: { Carousel, Slide, YearlyPlanPromotion, PlanBanners, PPage, PStack, PStackItem, PButton, PButtonGroup, PHeading, PLayout, PLayoutSection, PTextContainer, PDataTable, PDataTableCol, PDataTableRow, PIcon, PTextStyle, PCardSection, PCard, PSkeletonDisplayText, PSkeletonBodyText, PSkeletonPage, PEmptyState },
        props: ['shop_domain'],
        data() {
            return {
                slideLength : 0,
                perPage: 0,
                currentSlide: 0,
                planLoading: false,
                plan: {},
                plans: [],
                features: [],
                shopify_plan: '',
                default_plan_id: null,
                onboard: true,
                choose_later: false,
                has_active_charge: false,
                subtitleContent: '',
                checkList: [
                    "60 days free trial",
                ],
                selectedPlan:'monthly',
                monthlySelectedStyle:{
                    height: '60px',
                    backgroundColor:'#257f60',
                    color:'#fff',
                    position:'relative',
                    right:'-5px',
                    borderRadius:'8px',
                    zIndex: 1,
                },
                yearlySelectedStyle:{
                    height: '60px',
                    backgroundColor:'#257f60',
                    position:'relative',
                    left:'-5px',
                    borderRadius:'8px'
                },
                monthlyStyle:{
                    height: '55px',
                    backgroundColor:'#f0f8f5',
                    boxShadow: 'rgba(99, 99, 99, 0.2) 0px 2px 8px 0px',
                    border:'none',
                    borderRadius:'8px',
                    ZIndex: 11,
                },
                yearlyStyle:{
                    color:'#258060',
                    height: '55px',
                    backgroundColor:'#f0f8f5',
                    boxShadow: 'rgba(99, 99, 99, 0.2) 0px 2px 8px 0px',
                    border:'none',
                    borderRadius:'8px'
                },
            }
        },
        computed: {
            shop() {
                return {
                    "name": this.shop_domain,
                    "shopify_plan": this.shopify_plan,
                    "has_plan": !!this.plan,
                    "plan": this.plan,
                    "default_plan_id": this.default_plan_id,
                };
            },
            headings() {
                let headings = [('Plans & Features')];
                this.plans.forEach(plan => {

                    let heading = (plan.name);
                    if (plan.price > 0) heading += ` ($${plan.price}/mo)`;
                    headings.push(heading);
                });
                return headings;
            },
            monthlyPlan() {
                const plans = [];
                for(let planKey in this.plans) {
                    if(this.plans[planKey].interval === 'EVERY_30_DAYS') {
                        plans.push(this.plans[planKey]);
                    }
                }
                return plans;
            },
            yearlyPlan() {
                const plans = [];
                for(let planKey in this.plans) {
                    if(this.plans[planKey].interval === 'ANNUAL') {
                        plans.push(this.plans[planKey]);
                    }
                }
                return plans;
            },
        },
        methods: {
            handleNavigationClick($event) {
                const activeSlideIds = [];
                let activeSlides = document.getElementsByClassName('VueCarousel-slide-active')
                for (let i=0, max=activeSlides.length; i < max; i++) {
                    activeSlideIds.push(parseInt(activeSlides[i].id))
                }
                let last = activeSlideIds.pop();
                let first = last - (this.perPage - 1)

                let allSlides = document.getElementsByClassName('VueCarousel-slide');
                for (let i=0, max=allSlides.length; i < max; i++) {
                    let slide = document.getElementById(allSlides[i].id);
                    slide.classList.remove('first-slide')
                    slide.classList.remove('last-slide')
                }

                let firstSlideClassName = '.slide-' + first
                let element = document.querySelector(firstSlideClassName);
                element.classList.add('first-slide')

                let lastSlideClassName = '.slide-' + last
                element = document.querySelector(lastSlideClassName);
                element.classList.add('last-slide')

                if (document.querySelector('.VueCarousel-navigation-button.VueCarousel-navigation-prev')) {
                    document.querySelector('.VueCarousel-navigation-button.VueCarousel-navigation-prev').style.left = -document.querySelector('.Polaris-ResourceList__ResourceListWrapper.features').offsetWidth + 'px';
                }
            },
            activePlanStyle(plan) {
                return [plan.shopify_plans.includes(this.shop.shopify_plan) || !plan.store_base_plan ? {backgroundColor: '#f0f8f5', color: '#257f60'} : {}];
            },
            activePlanClass(plan) {
                return plan.shopify_plans.includes(this.shop.shopify_plan) || !plan.store_base_plan ? 'active-plan' : '';
            },
            isCurrentPlan(plan) {
                return this.has_active_charge && this.shop.plan && (plan.id === this.shop.plan.id || (!plan.is_custom && plan.base_plan === this.shop.plan.id));
            },
            isSamePlanInOtherInterval(plan) {
                return this.shop.plan && (plan.shopify_plans === this.shop.plan.shopify_plans)
            },
            format(feature) {
                if (['double', 'integer'].includes(feature?.value_type)) {
                    if (feature.format === 'percentage') {
                        return `${feature.value}%`
                    } else if (feature.format === 'count') {
                        return (feature.value < 0 ? (`Unlimited`) : feature.value)
                    } else return feature.value
                }
                else if(feature?.value_type === 'array') {
                    return JSON.parse(feature.value).join(' ')
                }
                else if(feature?.value_type === 'string') {
                    return feature.value.replace('"', '').replace('"', '')
                }
            },
            calculateDiscountedPrice(plan) {
                if (plan.discount_type === 'percentage') {
                    return plan.price - (plan.price * plan.discount)/100
                }
                else if (plan.discount_type === 'amount') {
                    return plan.price - plan.discount
                }
            },
            async getPlanUrl(plan) {
                let shopName = this.shop.name;
                const response = await axios.get(`${this.app_manager_config.baseUrl}/api/app-manager/plan/process/${plan.id}?shop=${shopName}`).catch(error => {
                    console.error(error)
                });
                if(response.data.plan_type === 'free_plan'){
                    this.$emit('handlePlanSelect', {free_plan: true})
                }else{
                    let redirectUrl = response.data.redirect_url;
                    if (redirectUrl) {
                        window.top.location.href = redirectUrl;
                    }
                }
            },
            async activePlan() {
                const response = await this.activeWithoutPlan()
                if (response.data.status === true && this.onboard) {
                    // Create the event
                    this.$emit('handlePlanSelect', {choose_later: true})
                    this.onboard = false;
                }
            },
            async activeWithoutPlan() {
                return await axios.post(`${this.app_manager_config.baseUrl}/api/app-manager/active-without-plan`, {
                    shop_domain: this.shop.name,
                    plan_id: this.shop.default_plan_id
                }).catch(error => {
                    console.error(error)
                });
            },
            async selectPlan(value){
                this.selectedPlan = value;
                this.$nextTick(() => {

                    let elements = document.querySelectorAll('.plan__price');
                    let maxHeight = 0;
                    elements.forEach((item) => {
                        if (maxHeight < item.offsetHeight) {
                            maxHeight = item.offsetHeight
                        }
                    });
                    elements.forEach((item) => {
                        item.style.minHeight = maxHeight + 'px';
                    });

                    this.slideLength = this.selectedPlan === 'monthly' ? this.monthlyPlan.length : this.yearlyPlan.length;
                    this.perPage = this.slideLength >= 4 ? 4 : this.slideLength;

                    // calculate and reset height of rows
                    this.features.forEach((feature) => {
                        let className = feature.value_type + '__type__' + feature.slug;
                        let elements = document.querySelectorAll('.' + className);
                        let maxHeight = 0;
                        elements.forEach((item) => {
                            item.style.minHeight = 'unset';
                            if (maxHeight < item.offsetHeight) {
                                maxHeight = item.offsetHeight
                            }
                        });
                        elements.forEach((item) => {
                            item.style.minHeight = maxHeight + 'px';
                        });
                    })

                    // remove first-slide and last-slide classes
                    let allSlides = document.getElementsByClassName('VueCarousel-slide');
                    for (let i=0, max=allSlides.length; i < max; i++) {
                        let slide = document.getElementById(allSlides[i].id);
                        slide.classList.remove('first-slide')
                        slide.classList.remove('last-slide')
                    }

                    // add first-slide and last-slide classes
                    let pagesCount = this.slideLength;
                    setTimeout(() => {
                        let element = document.querySelector('.slide-0');
                        if (element) {
                            element.classList.add('first-slide')
                            if (pagesCount < 4) {
                                let lastSlideClass = '.slide-' + (pagesCount-1)
                                element = document.querySelector(lastSlideClass);
                                element.classList.add('last-slide')
                            }
                            else {
                                element = document.querySelector('.slide-3');
                                element.classList.add('last-slide')
                            }
                            if (document.querySelector('.VueCarousel-navigation-button.VueCarousel-navigation-prev')) {
                                document.querySelector('.VueCarousel-navigation-button.VueCarousel-navigation-prev').style.left = -document.querySelector('.Polaris-ResourceList__ResourceListWrapper.features').offsetWidth + 'px';
                            }
                        }
                    }, 100)

                });
            },
            async fetchFeatures() {
                let {data} = await axios.get(`${this.app_manager_config.baseUrl}/api/app-manager/plan-features`).catch(error => {
                    console.error(error)
                });
                if (data.features.length) {
                    this.features = data.features;
                    this.features = this.features?.filter((item) => item.hidden_feature !== true)
                    this.features = this.features?.sort((featureA, featureB) => parseInt(featureA.display_order) - parseInt(featureB.display_order))
                }
            },
            async fetchPlans() {
                let {data} = await axios.get(`${this.app_manager_config.baseUrl}/api/app-manager/plans`, { params: { 'shop_domain': this.shop_domain } }).catch(error => {
                    console.error(error)
                });
                if (data.plans.length) {
                    this.plans = data.plans;
                    this.plans = this.plans?.sort((planA, planB) => parseFloat(planA.price) - parseFloat(planB.price));

                    if (this.plans[0].store_base_plan) {
                        this.subtitleContent = 'App plans are based on your existing shopify plan';
                    }

                    this.plan = data.plan;
                    if (this.plan?.interval === 'ANNUAL') {
                        this.selectedPlan = 'annually'

                    }
                    this.shopify_plan = data.shopify_plan;
                    this.default_plan_id = data.default_plan_id;
                    this.choose_later = data.choose_later;
                    this.onboard = this.default_plan_id && this.choose_later;
                    this.has_active_charge = data.has_active_charge;
                }
            },
            headerClasses(firstColumn) {
                return {
                    'Polaris-DataTable__Cell': true,
                    'Polaris-DataTable__Cell--header': true,
                    'Polaris-DataTable__Cell--verticalAlignMiddle': true,
                    'Polaris-DataTable__Cell--firstColumn': Boolean(firstColumn),
                };
            },
        },
        async mounted() {

            this.planLoading = true;
            await this.fetchFeatures();
            await this.fetchPlans();
            this.planLoading = false;

            this.$nextTick(() => {

                // calculate height of cell
                setTimeout(() => {

                    let elements = document.querySelectorAll('.plan__price');
                    let maxHeight = 0;
                    elements.forEach((item) => {
                        item.style.minHeight = 'unset';
                        if (maxHeight < item.offsetHeight) {
                            maxHeight = item.offsetHeight
                        }
                    });
                    elements.forEach((item) => {
                        item.style.minHeight = maxHeight + 'px';
                    });

                    this.features.forEach((feature) => {
                        let className = feature.value_type + '__type__' + feature.slug;
                        elements = document.querySelectorAll('.' + className);
                        maxHeight = 0;
                        elements.forEach((item) => {
                            if (maxHeight < item.offsetHeight) {
                                maxHeight = item.offsetHeight
                            }
                        });
                        elements.forEach((item) => {
                            item.style.minHeight = maxHeight + 'px';
                        });
                    })
                }, 100);

                // remove fist-slide and last-slide classes from all slides
                let allSlides = document.getElementsByClassName('VueCarousel-slide');
                for (let i=0, max=allSlides.length; i < max; i++) {
                    let slide = document.getElementById(allSlides[i].id);
                    slide.classList.remove('first-slide')
                    slide.classList.remove('last-slide')
                }

                // add first-slide and last-slide
                this.slideLength = this.selectedPlan === 'monthly' ? this.monthlyPlan.length : this.yearlyPlan.length;
                this.perPage = this.slideLength >= 4 ? 4 : this.slideLength;

                let pagesCount = this.slideLength;
                setTimeout(() => {
                    let element = document.querySelector('.slide-0');
                    if (element) {
                        element.classList.add('first-slide')
                        if (pagesCount < 4) {
                            let lastSlideClass = '.slide-' + (pagesCount-1)
                            element = document.querySelector(lastSlideClass);
                            element.classList.add('last-slide')
                        }
                        else {
                            element = document.querySelector('.slide-3');
                            element.classList.add('last-slide')
                        }
                        if (document.querySelector('.VueCarousel-navigation-button.VueCarousel-navigation-prev')) {
                            document.querySelector('.VueCarousel-navigation-button.VueCarousel-navigation-prev').style.left = -document.querySelector('.Polaris-ResourceList__ResourceListWrapper.features').offsetWidth + 'px';
                        }
                    }
                }, 100)

            });
        },
    }
</script>

<style lang="scss">

    @import url('https://fonts.googleapis.com/css2?family=Satisfy&display=swap');

    .app-manager .app-manager-plan-page-slider ul
    {
        list-style: none;
        margin: 0;
        padding: 0;
    }
    .app-manager .app-manager-plan-page-slider .Polaris-ResourceList__ResourceListWrapper.features li,
    .app-manager .app-manager-plan-page-slider .Polaris-Layout__Section .VueCarousel-slide li,
    .app-manager .app-manager-plan-page-slider .plan__price
    {
        padding: 16px 16px 16px 20px;
    }
    .app-manager .app-manager-plan-page-slider .Polaris-ResourceList__ResourceListWrapper.features li,
    .app-manager .app-manager-plan-page-slider .Polaris-Layout__Section .VueCarousel-slide li:not(:last-child),
    .app-manager .app-manager-plan-page-slider .Polaris-Layout.custom-plan .VueCarousel .plan__price
    {
        border-top: 1px solid #dddddd;
        border-right: 1px solid #dddddd;
        background: #fff;
    }
    .app-manager .app-manager-plan-page-slider .Polaris-ResourceList__ResourceListWrapper.features li{
        border-right: none;
        border-left: 1px solid #dddddd;
    }
    /*.app-manager .app-manager-plan-page-slider .plan__price{
        min-height:121px;
    }*/
    .app-manager .app-manager-plan-page-slider .Polaris-ResourceList__ResourceListWrapper.features li:last-child,
    .app-manager .app-manager-plan-page-slider .Polaris-Layout__Section .VueCarousel-slide li:nth-last-child(2)
    {
        border-bottom: 1px solid #dddddd;
    }
    .app-manager .app-manager-plan-page-slider .Polaris-ResourceList__ResourceListWrapper.features li:first-child
    {
        border-top-left-radius: 12px;
    }
    .app-manager .app-manager-plan-page-slider .Polaris-ResourceList__ResourceListWrapper.features li:last-child
    {
        border-bottom-left-radius: 12px;
    }
    .app-manager .app-manager-plan-page-slider .VueCarousel-inner .VueCarousel-slide.first-slide ul li:not(:last-child)
    {
        border-left: 1px solid #dddddd;
    }
    .app-manager .app-manager-plan-page-slider .VueCarousel-inner .VueCarousel-slide.first-slide .plan__price
    {
        border-left: 1px solid #dddddd;
        box-shadow: none;
        border-top-left-radius: 12px;
        overflow: hidden;
    }
    .app-manager .app-manager-plan-page-slider .VueCarousel-inner .VueCarousel-slide.last-slide ul li:nth-last-child(2)
    {
        border-bottom-right-radius: 12px;
    }
    .app-manager .app-manager-plan-page-slider .VueCarousel-inner .VueCarousel-slide.last-slide .plan__price
    {
        border-right: 1px solid #dddddd;
        box-shadow: none;
        border-top-right-radius: 12px;
        overflow: hidden;
    }
    .app-manager .app-manager-plan-page-slider .VueCarousel-inner .VueCarousel-slide.last-slide
    {
        border-top-right-radius: 12px;
    }
    .app-manager .app-manager-plan-page-slider .VueCarousel-inner .VueCarousel-slide.first-slide
    {
        border-top-left-radius: 12px;
    }
    .app-manager .app-manager-plan-page-slider .VueCarousel .VueCarousel-inner li
    {
        text-align: center;
    }
    .app-manager .app-manager-plan-page-slider .VueCarousel-navigation-button
    {
        color: #257f60;
    }
    .app-manager .app-manager-plan-page-slider .btn-group .Polaris-ButtonGroup__Item
    {
        margin-left: 0px !important;
        z-index: unset !important;
    }
    .app-manager .app-manager-plan-page-slider .feature__list,
    .app-manager .app-manager-plan-page-slider .feature__type__array
    {
        display: flex;
        align-items: center;
        word-spacing: 999px;
        justify-content: center;
    }
    .app-manager .app-manager-plan-page-slider .feature__type__array.feature__class
    {
        justify-content: left;
    }

</style>
