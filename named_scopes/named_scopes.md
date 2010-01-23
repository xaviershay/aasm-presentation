!SLIDE

# Named scopes

!SLIDE ruby

    @@@ruby
    # Turn this...
    named_scope :unpaid, :conditions => {:state => 'unpaid'}
    named_scope :paid,   :conditions => {:state => 'paid'}

!SLIDE ruby

    @@@ruby
    # Into this!
    class ActiveRecord::Base
      def self.named_scopes_for_states
        states.each do |state|
          named_scope state,
            :conditions => { :state => state.to_s }
        end
        named_scope :in_state, lambda {|state| {
          :conditions => { :state => state.to_s }}}
      end
    end

!SLIDE ruby

    @@@ruby
    class Order < ActiveRecord::Base
      acts_as_state_machine :initial => 'unpaid'

      state :unpaid
      state :paid

      named_scopes_for_states
    end
