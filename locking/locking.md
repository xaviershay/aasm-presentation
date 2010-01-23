!SLIDE

# Locking

!SLIDE ruby

    @@@ruby
    # WRONG
    def mark_order_as_paid
      @order.paid!
    end

!SLIDE ruby

    @@@ruby
    # Right!
    def mark_order_as_paid
      Order.transaction do
        @order.lock!
        @order.paid!
      end
    rescue ActiveRecord::RecordNotFound
      # say something nice to the user
    end

!SLIDE ruby smaller

    @@@ruby
    # DRY that!
    class ActiveRecord::Base
      def self.obtain_lock_before_transitions
        event_table.keys.each do |t|
          define_method("#{t}_with_lock!") do
            transaction do
              lock!
              send("#{t}_without_lock!")
            end
          end
          alias_method_chain "#{t}!", :lock
        end
      end
    end

!SLIDE ruby smaller

    @@@ruby
    class Order < ActiveRecord::Base
      acts_as_state_machine :initial => 'unpaid'

      state :paid

      event(:pay) do
        transitions :from => :unpaid, :to => :paid
      end

      obtain_lock_before_transitions
    end
